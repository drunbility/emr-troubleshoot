#### 问题现象

用户反馈他们客户端提交到 hs2 的 hos sql 任务提交成功后,一直输出 waiting 信息:
```
INFO: 2023-01-17 01:01:58:0158 INFO : Query ID = hadoop_20230117010148_ff66ccca-fee9-481b-b81c-25fae5e52532
INFO: 2023-01-17 01:01:58:0158 INFO : Total jobs = 3
INFO: 2023-01-17 01:01:58:0158 INFO : Launching Job 1 out of 3
INFO: 2023-01-17 01:01:58:0158 INFO : Starting task [Stage-1:MAPRED] in serial mode
INFO: 2023-01-17 01:02:05:0105 INFO : waiting for hadoop to execute ...
INFO: 2023-01-17 01:02:12:0112 INFO : waiting for hadoop to execute ...
INFO: 2023-01-17 01:02:20:0120 INFO : waiting for hadoop to execute ...
```

直到三小时后抛出了错误:
```
INFO: 2023-01-17 04:01:41:0141 INFO : waiting for hadoop to execute ...
INFO: 2023-01-17 04:01:51:0151 Taskname: load_dim_sku_course RunIndex: 20230116 failed due to queryengine error: Query Failed: org.apache.hive.service.cli.HiveSQLException: Error while processing statement: FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.spark.SparkTask. Failed to create spark client.
at org.apache.hive.service.cli.operation.Operation.toSQLException(Operation.java:380)
at org.apache.hive.service.cli.operation.SQLOperation.runQuery(SQLOperation.java:257)
at org.apache.hive.service.cli.operation.SQLOperation.access$800(SQLOperation.java:91)
at org.apache.hive.service.cli.operation.SQLOperation$BackgroundWork$1.run(SQLOperation.java:348)
at java.security.AccessController.doPrivileged(Native Method)
at javax.security.auth.Subject.doAs(Subject.java:422)
at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1844)
at org.apache.hive.service.cli.operation.SQLOperation$BackgroundWork.run(SQLOperation.java:362)
at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
at java.util.concurrent.FutureTask.run(FutureTask.java:266)
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
at java.lang.Thread.run(Thread.java:748)
Caused by: org.apache.hadoop.hive.ql.metadata.HiveException: Failed to create spark client.
at org.apache.hadoop.hive.ql.exec.spark.session.SparkSessionImpl.open(SparkSessionImpl.java:64)
at org.apache.hadoop.hive.ql.exec.spark.session.SparkSessionManagerImpl.getSession(SparkSessionManagerImpl.java:115)
at org.apache.hadoop.hive.ql.exec.spark.SparkUtilities.getSparkSession(SparkUtilities.java:126)
at org.apache.hadoop.hive.ql.exec.spark.SparkTask.execute(SparkTask.java:129)
at org.apache.hadoop.hive.ql.exec.Task.executeTask(Task.java:199)
at org.apache.hadoop.hive.ql.exec.TaskRunner.runSequential(TaskRunner.java:100)
at org.apache.hadoop.hive.ql.Driver.launchTask(Driver.java:2188)
at org.apache.hadoop.hive.ql.Driver.execute(Driver.java:1844)
at org.apache.hadoop.hive.ql.Driver.runInternal(Driver.java:1531)
at org.apache.hadoop.hive.ql.Driver.run(Driver.java:1242)
at org.apache.hadoop.hive.ql.Driver.run(Driver.java:1237)
at org.apache.hive.service.cli.operation.SQLOperation.runQuery(SQLOperation.java:255)
... 11 more
Caused by: java.lang.RuntimeException: java.util.concurrent.ExecutionException: java.util.concurrent.TimeoutException: Timed out waiting for client connection.
at com.google.common.base.Throwables.propagate(Throwables.java:160)
at org.apache.hive.spark.client.SparkClientImpl.<init>(SparkClientImpl.java:125)
at org.apache.hive.spark.client.SparkClientFactory.createClient(SparkClientFactory.java:89)
at org.apache.hadoop.hive.ql.exec.spark.RemoteHiveSparkClient.createRemoteClient(RemoteHiveSparkClient.java:101)
at org.apache.hadoop.hive.ql.exec.spark.RemoteHiveSparkClient.<init>(RemoteHiveSparkClient.java:97)
at org.apache.hadoop.hive.ql.exec.spark.HiveSparkClientFactory.createHiveSparkClient(HiveSparkClientFactory.java:73)
at org.apache.hadoop.hive.ql.exec.spark.session.SparkSessionImpl.open(SparkSessionImpl.java:62)
... 22 more
Caused by: java.util.concurrent.ExecutionException: java.util.concurrent.TimeoutException: Timed out waiting for client connection.
at io.netty.util.concurrent.AbstractFuture.get(AbstractFuture.java:41)
at org.apache.hive.spark.client.SparkClientImpl.<init>(SparkClientImpl.java:109)
... 27 more
Caused by: java.util.concurrent.TimeoutException: Timed out waiting for client connection.
at org.apache.hive.spark.client.rpc.RpcServer$2.run(RpcServer.java:172)
at io.netty.util.concurrent.PromiseTask$RunnableAdapter.call(PromiseTask.java:38)
at io.netty.util.concurrent.ScheduledFutureTask.run(ScheduledFutureTask.java:120)
at io.netty.util.concurrent.SingleThreadEventExecutor.runAllTasks(SingleThreadEventExecutor.java:399)
at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:464)
at io.netty.util.concurrent.SingleThreadEventExecutor$2.run(SingleThreadEventExecutor.java:131)
... 1 more
. errorCode: 1. sqlState: 08S01

```

#### 排查过程

首先跟踪报错的堆栈,`SparkSessionImpl.open`  开始,`TimeoutException`  抛出来的调用位置是
`at org.apache.hive.spark.client.SparkClientImpl.<init>(SparkClientImpl.java:125)`

看到 `SparkClientImpl` 的源码 125行,可以看到报错的调用是  `rpcServer.registerClient`

```
try {
      // The RPC server will take care of timeouts here.
      this.driverRpc = rpcServer.registerClient(clientId, secret, protocol).get();
    } catch (Throwable e) {
      if (e.getCause() instanceof TimeoutException) {
        LOG.error("Timed out waiting for client to connect.\nPossible reasons include network " +
            "issues, errors in remote driver or the cluster has no available resources, etc." +
            "\nPlease check YARN or Spark driver's logs for further information.", e);
      } else {
        LOG.error("Error while waiting for client to connect.", e);
      }
      driverThread.interrupt();
      try {
        driverThread.join();
      } catch (InterruptedException ie) {
        // Give up.
        LOG.debug("Interrupted before driver thread was finished.");
      }
      throw Throwables.propagate(e);
    }
```

这个方法内注册了一个超时时间:
```
SPARK_RPC_CLIENT_HANDSHAKE_TIMEOUT("hive.spark.client.server.connect.timeout",  
  "90000ms", new TimeValidator(TimeUnit.MILLISECONDS),  
  "Timeout for handshake between Hive client and remote Spark driver.  Checked by both processes."),
```

hs2 hive-site.xml 中这个超时时间被设置为了3个小时,所以用户侧,也就是提交客户端等待了三个小时.继续深入,这里为什么 hive sparkclient 跟 remote spark driver 握手会抛出这个超时错误

在客户端日志中找到 `Query ID` ,拿这个 id 去 hs2 日志中搜索,找到提交到 yarn 上的 application id ,
首先拿这个 app id  去 web ui 上看这个 application ,发现这个任务在 yarn 上早就已经失败了.所以上面 hive sparkclient 跟 remote spark driver 握手无法建立连接,直到三小时超时.

那为什么这个任务会判定为失败了?根据日志来分析整个链路,来熟悉 yarn 中的状态转换

去获取 yarn 日志, yarn logs applicationId  xxxx,发现获取到了空日志,说明 application master 并没有启动起来,只能去 rm 找信息.
接着拿着这个 app id 去 yarn 的 rm 日志中搜索,根据这个 app id 可以找到了第一个启动的 container,也就是 AM ,然后拿着这个 container id 在 rm 日志中搜索,观察 am 的状态转换过程,找到关键信息:

![[Pasted image 20230117203825.png]]

根据日志显示时间看,这里是10分钟后,rm 判定  am 心跳超时了,am 心跳超时一般是 am 的内存不够导致的,没有汇报上来.

这个时间刚好跟
`yarn.am.liveness-monitor.expiry-interval-ms`  
配置的默认值 10 分钟对得上.rm 没有在10分钟内得到 am 的状态,判定这个任务为 failed 状态了,接着 rm 会向 nodemanager 发起 kill am container 的请求,查看 nm 的日志,可以看到这个请求:

![[Pasted image 20230117204332.png]]


最终 rm 将这个 application 判定为 failed 状态了:
![[Pasted image 20230117204726.png]]

注意到最后打印的失败原因, am 启动超时了,并且只尝试了一次 `global limit =2; local limit is =1`

这里 limit 就是 am 启动的重试次数,默认是 2, yarn.resourcemanager.am.max-attempts  配置的默认值 . local limit 是由于任务提交设置了参数 spark.yarn.maxAppAttempts =1  ,所以覆盖了上面的值


 



