

### 现象

用户反馈他们查看 hiveserver2 的日志打印,发现打印的和 hms 的连接数非常高,数目有明显异常,实际并没有这么多

![[wecom-temp-130359-bfcc314cdea4ea76566c70cb8a9b8f02.png]]


### 排查过程

登陆 hs2 机器,查看日志,确实连接数打印异常高,接着查看日志,看到 warn 信息

```
2022-03-29T11:29:27,379  WARN [HiveServer2-Handler-Pool: Thread-1214391] transport.TIOStreamTransport: Error closing output stream.
java.net.SocketException: Socket closed
        at java.net.SocketOutputStream.socketWrite(SocketOutputStream.java:118) ~[?:1.8.0_252]
        at java.net.SocketOutputStream.write(SocketOutputStream.java:155) ~[?:1.8.0_252]
        at java.io.BufferedOutputStream.flushBuffer(BufferedOutputStream.java:82) ~[?:1.8.0_252]
        at java.io.BufferedOutputStream.flush(BufferedOutputStream.java:140) ~[?:1.8.0_252]
        at java.io.FilterOutputStream.close(FilterOutputStream.java:158) ~[?:1.8.0_252]
        at org.apache.thrift.transport.TIOStreamTransport.close(TIOStreamTransport.java:110) [hive-exec-2.3.7.jar:2.3.7]
        at org.apache.thrift.transport.TSocket.close(TSocket.java:235) [hive-exec-2.3.7.jar:2.3.7]
        at org.apache.hadoop.hive.metastore.HiveMetaStoreClient.close(HiveMetaStoreClient.java:576) [hive-exec-2.3.7.jar:2.3.7]
        at sun.reflect.GeneratedMethodAccessor50.invoke(Unknown Source) ~[?:?]
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[?:1.8.0_252]
        at java.lang.reflect.Method.invoke(Method.java:498) ~[?:1.8.0_252]
        at org.apache.hadoop.hive.metastore.RetryingMetaStoreClient.invoke(RetryingMetaStoreClient.java:173) [hive-exec-2.3.7.jar:2.3.7]
        at com.sun.proxy.$Proxy47.close(Unknown Source) [?:?]
        at sun.reflect.GeneratedMethodAccessor50.invoke(Unknown Source) ~[?:?]
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[?:1.8.0_252]
        at java.lang.reflect.Method.invoke(Method.java:498) ~[?:1.8.0_252]
        at org.apache.hadoop.hive.metastore.HiveMetaStoreClient$SynchronizedHandler.invoke(HiveMetaStoreClient.java:2450) [hive-exec-2.3.7.jar:2.3.7]
        at com.sun.proxy.$Proxy47.close(Unknown Source) [?:?]
        at org.apache.hadoop.hive.ql.metadata.Hive.close(Hive.java:409) [hive-exec-2.3.7.jar:2.3.7]
        at org.apache.hadoop.hive.ql.metadata.Hive.access$000(Hive.java:169) [hive-exec-2.3.7.jar:2.3.7]
        at org.apache.hadoop.hive.ql.metadata.Hive$1.remove(Hive.java:190) [hive-exec-2.3.7.jar:2.3.7]
        at org.apache.hadoop.hive.ql.metadata.Hive.closeCurrent(Hive.java:376) [hive-exec-2.3.7.jar:2.3.7]
        at org.apache.hadoop.hive.ql.session.SessionState.close(SessionState.java:1592) [hive-exec-2.3.7.jar:2.3.7]
        at org.apache.hive.service.cli.session.HiveSessionImpl.close(HiveSessionImpl.java:739) [hive-service-2.3.7.jar:2.3.7]
        at org.apache.hive.service.cli.session.SessionManager.closeSession(SessionManager.java:448) [hive-service-2.3.7.jar:2.3.7]
        at org.apache.hive.service.cli.CLIService.closeSession(CLIService.java:239) [hive-service-2.3.7.jar:2.3.7]
        at org.apache.hive.service.cli.thrift.ThriftCLIService.CloseSession(ThriftCLIService.java:490) [hive-service-2.3.7.jar:2.3.7]
        at org.apache.hive.service.rpc.thrift.TCLIService$Processor$CloseSession.getResult(TCLIService.java:1397) [hive-exec-2.3.7.jar:2.3.7]
        at org.apache.hive.service.rpc.thrift.TCLIService$Processor$CloseSession.getResult(TCLIService.java:1382) [hive-exec-2.3.7.jar:2.3.7]
        at org.apache.thrift.ProcessFunction.process(ProcessFunction.java:39) [hive-exec-2.3.7.jar:2.3.7]
        at org.apache.thrift.TBaseProcessor.process(TBaseProcessor.java:39) [hive-exec-2.3.7.jar:2.3.7]
        at org.apache.hive.service.auth.TSetIpAddressProcessor.process(TSetIpAddressProcessor.java:56) [hive-service-2.3.7.jar:2.3.7]
        at org.apache.thrift.server.TThreadPoolServer$WorkerProcess.run(TThreadPoolServer.java:286) [hive-exec-2.3.7.jar:2.3.7]
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [?:1.8.0_252]
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [?:1.8.0_252]
        at java.lang.Thread.run(Thread.java:748) [?:1.8.0_252]

```

按照上面堆栈,找到源码位置:

`org.apache.hadoop.hive.metastore.HiveMetaStoreClient#close`

看代码逻辑,这里 decrement 逻辑明显是有可能是没有走到的

![[wecom-temp-255604-a1fe4f2a847e2f6f8dd93b4ffc93a382.png]]


### 结论
按照 [[hive 执行 desc table 较慢问题#^92768d]]  中比较 git 分支方法,在最新的 master 分支上查看该类的这个方法,查看提交记录,发现这里确定存在问题,社区已经提了修复办法:

[[HIVE-24349] Client connection count is not printed correctly in HiveMetastoreClient - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/HIVE-24349)

### 相关问题
关于 hs2 日志中打印连接数不准确的问题,还有一种场景,就是 [[HiveServer2日志打印 session count 不准确问题]]




