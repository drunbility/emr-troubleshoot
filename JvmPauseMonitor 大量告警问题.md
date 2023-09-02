用户多次反馈 hdfs namenode 日志中出现大量告警信息

`org.apache.hadoop.util.JvmPauseMonitor: Detected pause in JVM or host machine (eg GC): pause of approximately`

最开始怀疑是由于jvm 发生了 [[jvm gc|full gc]] 以及机器的磁盘 io 打满导致的.但排查进程的 gc 日志,并没有发现频繁的 full gc 记录,并且 gc 日志记录跟 JvmPauseMonitor 告警时间对不上.
接着排查磁盘的 iowait 情况,iowait 高的情况也跟 JvmPauseMonitor 告警信息时间无法一一对应.
这两个猜测都没有明确的证据来支持


接着排查发现 pause 告警信息是规律性的每两分钟出现一次,接着内部对齐信息,我们机器上的 agent 会每2分钟使用 [[java 命令行工具]]  `jmap -histo` 去采集进程的对象分布信息 ,问题原因最终找到:

采集的命令会触发jvm强制所有线程进入saftpoint,safepoint 也会造成 STW  而不同的线程进入的时间不一致  木桶效应  最晚进入的时间就是最早进入的线程的停顿时间.  -XX:-UseBiasedLocking   XX:+UseCountedLoopSafepoints  这两个参数可以缩短进入safepoint的时间

最终把 agent 采集 jmap 命令去掉后, pause 告警日志消息,问题解决


---
[JVM Pauses - It's more than GC (blanco.io)](https://blanco.io/blog/jvm-safepoint-pauses/)









