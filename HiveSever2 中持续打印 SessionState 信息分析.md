

### 问题现象

用户反馈多台 hs2 ,有台 hs2 的日志持续快速打印 SessionState update 线程名称信息
```
2023-01-30T20:28:56,176  INFO [HiveServer2-Handler-Pool: Thread-185] conf.HiveConf: Using the default value passed in for log id: 87698bf3-4657-49ca-9015-c98d2f76bf55
2023-01-30T20:28:56,176  INFO [HiveServer2-Handler-Pool: Thread-185] session.SessionState: Updating thread name to 87698bf3-4657-49ca-9015-c98d2f76bf55 HiveServer2-Handler-Pool: Thread-185
2023-01-30T20:28:56,182  INFO [9d1056c4-5b83-4b73-9b6e-bedd7632f121 HiveServer2-Handler-Pool: Thread-187] conf.HiveConf: Using the default value passed in for log id: 9d1056c4-5b83-4b73-9b6e-bedd7632f121
2023-01-30T20:28:56,182  INFO [9d1056c4-5b83-4b73-9b6e-bedd7632f121 HiveServer2-Handler-Pool: Thread-187] session.SessionState: Resetting thread name to  HiveServer2-Handler-Pool: Thread-187
2023-01-30T20:28:56,182  INFO [87698bf3-4657-49ca-9015-c98d2f76bf55 HiveServer2-Handler-Pool: Thread-185] conf.HiveConf: Using the default value passed in for log id: 87698bf3-4657-49ca-9015-c98d2f76bf55
2023-01-30T20:28:56,182  INFO [87698bf3-4657-49ca-9015-c98d2f76bf55 HiveServer2-Handler-Pool: Thread-185] session.SessionState: Resetting thread name to  HiveServer2-Handler-Pool: Thread-185
2023-01-30T20:28:56,214  INFO [HiveServer2-Handler-Pool: Thread-185] conf.HiveConf: Using the default value passed in for log id: 87698bf3-4657-49ca-9015-c98d2f76bf55
2023-01-30T20:28:56,214  INFO [HiveServer2-Handler-Pool: Thread-185] session.SessionState: Updating thread name to 87698bf3-4657-49ca-9015-c98d2f76bf55 HiveServer2-Handler-Pool: Thread-185
2023-01-30T20:28:56,216  INFO [HiveServer2-Handler-Pool: Thread-187] conf.HiveConf: Using the default value passed in for log id: 9d1056c4-5b83-4b73-9b6e-bedd7632f121
2023-01-30T20:28:56,216  INFO [HiveServer2-Handler-Pool: Thread-187] session.SessionState: Updating thread name to 9d1056c4-5b83-4b73-9b6e-bedd7632f121 HiveServer2-Handler-Pool: Thread-187
2023-01-30T20:28:56,220  INFO [87698bf3-4657-49ca-9015-c98d2f76bf55 HiveServer2-Handler-Pool: Thread-185] conf.HiveConf: Using the default value passed in for log id: 87698bf3-4657-49ca-9015-c98d2f76bf55
2023-01-30T20:28:56,220  INFO [87698bf3-4657-49ca-9015-c98d2f76bf55 HiveServer2-Handler-Pool: Thread-185] session.SessionState: Resetting thread name to  HiveServer2-Handler-Pool: Thread-185
2023-01-30T20:28:56,222  INFO [9d1056c4-5b83-4b73-9b6e-bedd7632f121 HiveServer2-Handler-Pool: Thread-187] conf.HiveConf: Using the default value passed in for log id: 9d1056c4-5b83-4b73-9b6e-bedd7632f121
2023-01-30T20:28:56,222  INFO [9d1056c4-5b83-4b73-9b6e-bedd7632f121 HiveServer2-Handler-Pool: Thread-187] session.SessionState: Resetting thread name to  HiveServer2-Handler-Pool: Thread-187

...

```

用户觉得对比其他 hs2 ,这台 hs2 有异常,需要解释为什么会打印这种日志信息



### 排查过程

首先排查上面这些日志信息都是 info 级别,也就是说是由于正常的业务查询造成的,但是什么样的业务查询会导致 hs2 持续打印这些信息了?

从源码看,日志重复打印是在两个地方: `org.apache.hadoop.hive.ql.session.SessionState#updateThreadName`
`org.apache.hadoop.hive.ql.session.SessionState#resetThreadName`



向上查找, 这个方法是被这个方法调用 `org.apache.hive.service.cli.session.HiveSessionImpl#acquire`
`org.apache.hive.service.cli.session.HiveSessionImpl#release`



接着发现这个方法会把很多地方调用 ,`HiveSessionImpl` 内的很多操作,比如 execute statement ,get catalogs, get tables 很多操作都会调用这个 `acquire` 方法,难道客户端在不停重复这些操作吗?

这个时候去抓取 hs2 的线程栈,发现如下线程:

![[企业微信截图_4f3f17f5-1ec5-4937-b534-1b1294625875.png]]

看到最底下的方法 `org.apache.hive.service.cli.session.HiveSessionImpl#fetchResults`

这个方法确实会调用  `acquire` 方法,那么是由于这个 session 不停在 fetch lresut ,然后导致日志中不停打印信息吗?


### 结论

分析当时日志,发现在持续打印这些信息前, hs2 里面是执行了一个 sql

`select * from view_dws_lpc_leads_course_link_d where dt = '20230121'`

结合 `HiveSessionImpl` 的源码找到原因:

如果执行上述查询返回较多数据的话，会进行多次fetch result，每次返回`hive.server2.thrift.resultset.default.fetch.size`，默认为1000条：有如下调用链路 fetchResult->getOperationNextRowSet。

在fetchResult调用中，会在finally中调用release函数，在release函数中会将session thread name进行reset。

其中，当进行reset threadname时，会将带有sessionid的name恢复成不带sessionid。所有，当执行上述查询语句时，会在同一个session中不断fetch result，进而出现日志中现象。
同时，日志中涉及的 sessionid 也对应到了上面这种不带limit的select查询。

这个查询会返回非常多的数据给客户端,所以 hs2 会持续的执行 fetch result 方法返回数据给客户端,从而持续的打印上述日志



