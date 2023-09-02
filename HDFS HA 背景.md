在Hadoop 1.0时代，HDFS的架构相对简单，组件单一，主要包含NameNode，Seconday NameNode，DataNode和DFSClient四个核心组件，其中NameNode提供元数据管理服务，Secondary NameNode以冷备的状态为NameNode分担Checkpoint工作（定期合并FsImage和Editlog) 。在这种架构下NameNode是整个系统的单点，一旦出现故障对可用性是致命的。

SecondaryNameNode是namenode的冷备份，而namenode的HA高可用才是namenode的热备份。区别是SecondaryNameNode中存储的元数据不是实时的，滞后于namenode主节点.


SecondaryNameNode 依然存在单点故障(SPOF)问题,社区为了解决这个问题,中间尝试过多种 HA 方案,最终使用 HA using Quorum Journal Manager（QJM） 方法,并在 hadoop 2.0 版本中发布


中间社区也尝试了其他方法,比如 checkpoint node ,backup node   , 这些方案目前也在 hdfs namenode 命令中保留下来了

![[Pasted image 20221126152027.png]]

checkpoint node 和 secondary node 类似,唯一区别是 checkpoint node 会把合并后的 fsimage 上传到 namenode .

backup node 功能和 checkpoint node 类似 , 但是有一个重要的不同点是 backup node 也会在内存中维护一份 namespace ,并通过读取namenode edits 流来保持和 namenode 内存一致,所以生成 checkpoint 不需要从 namenode 上拉取 fsimage 和 edits files   
关于 checkpoint 和 backup node 的设计可以参考 [HADOOP-4539](https://issues.apache.org/jira/browse/HADOOP-4539)

如今,在生产环境中,一般使用的各种方案的集大成者 [[HA Using QJM]] 方案


---
[HDFS HA Using QJM原理解析 - Hexiaoqiao](https://hexiaoqiao.github.io/blog/2018/03/30/the-analysis-of-basic-principle-of-hdfs-ha-using-qjm/)

