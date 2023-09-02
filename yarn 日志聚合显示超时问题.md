### 问题现象

yarn 任务失败了,但是 web ui 上显示日志聚合超时了
![[Pasted image 20230418175815.png]]



### 排查过程

日志聚合超时是正常现象,因为由于 yarn 上的任务繁忙,导致某些任务在规定的时间内没有完成日志聚合,所以 resourcemanager 把这些任务的日志聚合标记为超时状态了.这个超时时间可以通过参数
`yarn.log-aggregation-status.time-out.ms` 配置


实际上最终该任务的日志是聚合成功的,可以在日志聚合目录
`yarn.nodemanager.remote-app-log-dir`  配置指定目录下面找到对应 app id 的日志目录

