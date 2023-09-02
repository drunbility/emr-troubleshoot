
emr集群中hdfs的日志级别为WARN，如果定位问题时需要更精细的日志级别，则可以使用动态的方式开启hdfs的日志，无需重启进程，可以做到随用随开，用完随关



查询当前类日志级别：
```
hadoop daemonlog -getlevel ip:4008 org.apache.hadoop.hdfs.server.namenode.FSImage
设置类的日志级别：
hadoop daemonlog -setlevel ip:4008 org.apache.hadoop.hdfs.server.namenode.FSImage DEBUG

```

也可以在 web ui 上改:
`http://RM_IP:port/logLevel`
`http://NN_IP:port/logLevel`

edits加载问题 
org.apache.hadoop.hdfs.server.namenode.FSEditLogLoader  org.apache.hadoop.hdfs.server.namenode.FSImage

NN块申请失败 org.apache.hadoop.hdfs.server.blockmanagement.BlockPlacementPolicyDefault





