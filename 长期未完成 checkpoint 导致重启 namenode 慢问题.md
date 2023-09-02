

对于大规模的集群,如果长期未成功完成 checkpoint ,那么会积累非常多的 editlog 文件.重启 namenode 的时候,必须要回放 editlog ,以使内存中的目录树恢复到最新状态.回放 editlog 必然是逐个文件来回放的,此时如果积累了大量的 editlog 文件,那么这个过程会长达三个小时以上.增大  namenode 的内存可以适当加快这个过程.


###  edit 文件堆积处理

如果长期 edit 日志文件有堆积,可以进入安全模式后,手动运行 saveNamespace 命令来进行一次合并. 但是线上环境中,不能进入安全模式,这个时候可以通过重启 standynamenode 来触发一次 checkpoint 

遇到过一次线上问题,由于 ann 锁的问题导致 sbnn 无法 put fsimage 到 ann,重启 sbnn 也无法完成最终完成 checkpoint ,这个时候可以等  sbnn namespace 正常启动后,然后进行一次主备切换,使之前锁住的 ann 变成了 sbnn,然后重启这个节点,此时就能够完成  checkpoint 了,堆积的 edit 文件也能够被清理了

### 手动 checkpoint 方法

^d799b1

hdfs dfsadmin -fs 10.0.0.26:4007 -safemode enter
hdfs dfsadmin -fs 10.0.0.26:4007 -saveNamespace
hdfs dfsadmin -fs 10.0.0.26:4007 -safemode leave
hdfs dfsadmin -safemode forceExit  //强制退出安全模式  

### 监控 checkpoint 

有两个重要指标:
1. TransactionsSinceLastCheckpoint 表示距离上次checkpoint的事务数，建议超过3000000 告警
2. LastCheckpointTime  上次checkpoint的时间，建议距离当前时间超过12小时告警


### edit logs 错误导致启动失败

hdfs 重启最常见的问题是由于各种 [[ edit logs 错误导致 hdfs 无法重启]]

