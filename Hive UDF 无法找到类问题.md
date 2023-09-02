### 现象

用户反馈他们使用 udf 时, 会报找不到类错误
```
2022-03-23T09:45:09,924  WARN [HiveServer2-Handler-Pool: Thread-472] thrift.ThriftCLIService: Error executing statement:
org.apache.hive.service.cli.HiveSQLException: Error running query: java.lang.NoClassDefFoundError: com/xxx/hiveudf/json/ToJsonUDF$IntInspectorHandle
	at org.apache.hive.service.cli.operation.SQLOperation.prepare(SQLOperation.java:238) ~[hive-service-2.3.7.jar:2.3.7]
	at org.apache.hive.service.cli.operation.SQLOperation.runInternal(SQLOperation.java:290) ~[hive-service-2.3.7.jar:2.3.7]
	at org.apache.hive.service.cli.operation.Operation.run(Operation.java:320) ~[hive-service-2.3.7.jar:2.3.7]
	at org.apache.hive.service.cli.session.HiveSessionImpl.executeStatementInternal(HiveSessionImpl.java:531) ~[hive-service-2.3.7.jar:2.3.7]
	at org.apache.hive.service.cli.session.HiveSessionImpl.executeStatementAsync(HiveSessionImpl.java:518) ~[hive-service-2.3.7.jar:2.3.7]
	at org.apache.hive.service.cli.CLIService.executeStatementAsync(CLIService.java:310) ~[hive-service-2.3.7.jar:2.3.7]
	at org.apache.hive.service.cli.thrift.ThriftCLIService.ExecuteStatement(ThriftCLIService.java:530) [hive-service-2.3.7.jar:2.3.7]
	at org.apache.hive.service.rpc.thrift.TCLIService$Processor$ExecuteStatement.getResult(TCLIService.java:1437) [hive-exec-2.3.7.jar:2.3.7]
	at org.apache.hive.service.rpc.thrift.TCLIService$Processor$ExecuteStatement.getResult(TCLIService.java:1422) [hive-exec-2.3.7.jar:2.3.7]
	at org.apache.thrift.ProcessFunction.process(ProcessFunction.java:39) [hive-exec-2.3.7.jar:2.3.7]
	at org.apache.thrift.TBaseProcessor.process(TBaseProcessor.java:39) [hive-exec-2.3.7.jar:2.3.7]
	at org.apache.hive.service.auth.TSetIpAddressProcessor.process(TSetIpAddressProcessor.java:56) [hive-service-2.3.7.jar:2.3.7]
	at org.apache.thrift.server.TThreadPoolServer$WorkerProcess.run(TThreadPoolServer.java:286) [hive-exec-2.3.7.jar:2.3.7]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [?:1.8.0_252]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [?:1.8.0_252]
	at java.lang.Thread.run(Thread.java:748) [?:1.8.0_252]
```

确认这个 udf  jar 里面有这个类 ，在HiveServer就会报这个错，在hiveclient执行没问题

在hiveserver端要运行下 reload function 后才能正常使用,并且每次重连都需要执行

### 排查过程


这里肯定是 classloader 出现问题了,或者是出现类冲突了,临时解决办法是将 udf jar 放到 hive auxlibe 下面来永久性的加载到 jvm 中,验证这种办法是可行的

分析到的情况是sesssio1 的classload某些情况下会污染session2的classload，都在某些类无法加载，通过auxlib可以保证放进去的jar一定会加载



### 结论

分析确认为 ranger  hive  plugin 的社区 bug:

https://issues.apache.org/jira/browse/RANGER-2376  

在测试环境通过打ranger patch验证问题解决.
