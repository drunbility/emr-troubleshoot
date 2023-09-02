
##### 问题

hive on spark 任务提交侧报错

`Caused by: java.lang.IllegalStateException: RPC channel is closed.`

#### 排查过程

在客户端日志中可以找到 query id ,然后拿  query id 去  hs2 日志中找到记录,接着根据 thread name 查找任务流程`Thread-46869`
看到日志中也只打印了上述信息:
`Caused by: java.lang.IllegalStateException: RPC channel is closed.`

根据日志信息,是 `RemoteHiveSparkClient` 这个方法里面抛出了错误

![[企业微信截图_b87f9112-cd7f-47e2-9b9f-a25816ef2878.png]]


根据原理,这种情况是 yarn 对端关闭了这个 rpc channel ,但是由于 spark submit 提交失败了,所以 hs2 driver 最终没有得到这个查询的 yarn appliction id.
但是为什么 yarn 对端关闭了这个 RPC channel 的原因,还是需要去 yarn 上找.

这个时候观察到客户端报错信息中输出了 task name

`INFO: 2023-02-09 11:45:17:0217 Taskname: f_bbe380f0d29211ec8d0ca17d86bde24a RunIndex: 20230208 failed due to queryengine error: Query Failed: org.apache.hive.service.cli.HiveSQLException: Error while processing statement: FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.spark.SparkTask. Failed to submit Spark work, please retry later`


那这个task name 去 hs2 日志中搜,发现了这个 task name 实际上是  app name

`2023-02-09T11:40:28,271  INFO [HiveServer2-Background-Pool: Thread-46869] spark.HiveSparkClientFactory: load spark property from hive configuration (spark.app.name -> baiyuxin01#zuoye#f_bbe380f0d29211ec8d0ca17d86bde24a#20230208.6).`

这个 app name 应该是直接发送给了 yarn rm 的,于是在 yarn 的 rm 日志搜索这个 app name ,最终找到这个任务的 application id ,接着去 yarn ui 上找这个 application id ,最终找到任务报错原因:
由于内存设置不够,导致 application master 无法启动,导致 hs2 中抛出了  `RPC channel is closed` 异常


#### 总结

hive on spark 任务 hs2 中 driver 提交任务 spark submit 失败没有取到 app id, 这个时候要去关联 yarn 上的信息,可以通过 app name 去关联

