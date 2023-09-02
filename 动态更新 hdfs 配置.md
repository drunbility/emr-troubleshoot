
操作步骤

1. 列出所有可以 reconfig 来即时生效的配置
	hdfs dfsadmin -reconfig namenode masterip:4007 properties  

2. 修改配置文件 hdfs-site.xml 中配置

3. 使新配置生效
	hdfs dfsadmin -reconfig namenode NN ip:4007 start  

4. 查看更新配置是否成功
	hdfs dfsadmin -reconfig namenode NN ip:4007 status  



使用场景

可以使用 reconfig 命令来做热换盘[更新 dfs.datanode.data.dir 目录](https://hadoop.apache.org/docs/r2.8.5/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html#DataNode_Hot_Swap_Drive),或者配置 [[hdfs balancer 加速]]


