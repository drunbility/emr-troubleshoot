#### 问题现象
用户反馈他们一个 hive on spark 任务卡主不动,在 spark web ui 上能看到 job 都完成了,但 application 无法结束,并且客户端持续打印日志:

`INFO : waiting for hadoop to execute ...`


![[Pasted image 20230118112800.png]]



#### 排查过程

开始排查没有任何头绪,这时用户反馈有个 hs2 告警连接数打满了,并且现场抓取 jstack 信息,显示有一个死锁:

![[wecom-temp-86880-9591bc1fd2666296241279c3a3c14cfb.jpg]]

由于业务比较紧急,所以先重启 hs2 ,解除死锁,恢复 hs2 对外服务. 重启 hs2 后, hs2 连接数恢复,同时任务重跑后成功.

排查具体根因,用户提供信息,发现跑成功的这个任务是有4个 job 的:

![[wecom-temp-250076-bf4144509fb30780cbcf5fbfed1eae13.jpg]]


根据这个信息,去异常卡住的 yarn app 日志里面找 am container 也就是 spark driver 的信息,跟踪调度的信息,发现 job 应该是串行调度执行的,当执行完 job 1 后,马上开始执行下一个 job:

![[企业微信截图_6b0f3a77-a7f4-442b-8815-3abd8ee736ea.png]]

但是在执行 job2 后, driver 一直没有收到新的 job request :

![[企业微信截图_70b69698-7cf5-4b2e-8a2e-59dff4ac53a6.png]]

这里就能够解释  spark ui 上看到的 3个 spark job 已经完成,但是卡住的现象了.但是为什么 spark driver 一直没有收到第四个 job  

这个就跟 hiveserver2 中 Driver 有关了,我们看到在 hive 中的 Driver 的 `execute(boolean deferClose)` 方法中:

```
// Add root Tasks to runnable  
for (Task<? extends Serializable> tsk : plan.getRootTasks()) {  
  // This should never happen, if it does, it's a bug with the potential to produce  
  // incorrect results.  assert tsk.getParentTasks() == null || tsk.getParentTasks().isEmpty();  
  driverCxt.addToRunnable(tsk);  
  
  if (metrics != null) {  
    tsk.updateTaskMetrics(metrics);  
  }  
}  
  
perfLogger.PerfLogBegin(CLASS_NAME, PerfLogger.RUN_TASKS);  
// Loop while you either have tasks running, or tasks queued up  
while (driverCxt.isRunning()) {  
  
  // Launch upto maxthreads tasks  
  Task<? extends Serializable> task;  
  while ((task = driverCxt.getRunnable(maxthreads)) != null) {  
    TaskRunner runner = launchTask(task, queryId, noName, jobname, jobs, driverCxt);  
    if (!runner.isRunning()) {  
      break;  
    }  
  }

```

最终生成的执行计划是一个 task 树(也可以说是 hive stage ),并且对于 Hos 来说,这个树是串行执行的,一般来说 MR task 可以并行执行 stage/task ,由参数 `hive.exec.parallel` 开关, 但在 `launchTask` 方法内判定 Hos 任务为串行执行的

### 问题结论
所以,当时的情况是 hs2 中该任务的 hs2 session 的 driver 在启动前三个 task 后,发生[[ java 死锁| 死锁]]了,
导致这个 driver 一直在等待,无法 launch 第四个 task .所以这个 session 的远端客户端在持续打印日志:
`INFO : waiting for hadoop to execute ...`

另一端 yarn am spark driver 一直在等待 driver 这边发送新的 task 请求或者结束 application 请求


