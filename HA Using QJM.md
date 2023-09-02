
### 架构
HDFS HA using QJM方案在2012年随Hadoop 2.0正式发布，此后一直是Hadoop社区默认HDFS HA方案。

![[Pasted image 20221126170018.png]]



从上图中，我们可以看出 NameNode 的高可用架构主要分为下面几个部分：

Active NameNode 和 Standby NameNode：两台 NameNode 形成互备，一台处于 Active 状态，为主 NameNode，另外一台处于 Standby 状态，为备 NameNode，只有主 NameNode 才能对外提供读写服务。Active NN节点通过QJM组件来同步 EditLog 文件给 Standby NN,Standby 节点定时checkpoint生成Fsimage文件，并通过http协议传给Active NN节点。

主备切换控制器 ZKFailoverController：ZKFailoverController 作为独立的进程运行，对 NameNode 的主备切换进行总体控制。ZKFailoverController 能及时检测到 NameNode 的健康状况，在主 NameNode 故障时借助 Zookeeper 实现自动的主备选举和切换，当然 NameNode 目前也支持不依赖于 Zookeeper 的手动主备切换。

Zookeeper 集群：为主备切换控制器提供主备选举支持。

共享存储系统：共享存储系统是实现 NameNode 的高可用最为关键的部分，共享存储系统保存了 NameNode 在运行过程中所产生的 HDFS 的元数据。主 NameNode 和NameNode 通过共享存储系统实现元数据同步。在进行主备切换的时候，新的主 NameNode 在确认元数据完全同步之后才能继续对外提供服务，主要有JournalNode 。

DataNode 节点：除了通过共享存储系统共享 HDFS 的元数据信息之外，主 NameNode 和备 NameNode 还需要共享 HDFS 的数据块和 DataNode 之间的映射关系。DataNode 会同时向主 NameNode 和备 NameNode 上报数据块的位置信息。


### 故障转移
当Active NN故障时，Zookeeper创建的临时节点 [[active nn hang 死导致全部进入 standby 问题|ActiveStandbyElectorLock]]将要被删除，其他NN节点注册的Watcher 来监听到该变化，NN节点的 ZKFailoverController 会马上再次进入到创建/hadoop-ha/${dfs.nameservices}/ActiveStandbyElectorLock 节点的流程，如果创建成功，这个本来处于 Standby 状态的 NameNode 就选举为主 NameNode 并随后开始切换为 Active 状态。

新当选的Active NN将确保从QJM(Quorum Journal Manager)同步完所有的元数据文件EditLog文件，然后切换为主节点，并向外提供服务。

hdfs 提供了 [haadmin 命令](https://hadoop.apache.org/docs/r2.8.5/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithQJM.html#Administrative_commands) 来进行手动管理

进行手动管理时候,有两个重要的细节:
1. 依据 dfs.ha.automatic-failover.enabled  是否设置为 true ,[[hdfs 主备状态手动切换|手动切换主备状态会有不同的行为]]
2. 为了防止出现两个 active namenode 情况(脑裂), namenode 提供了 [[ hdfs fence]] 机制

### QJM故障
JournalNode集群,默认三个节点。
1.当一个节点故障时，集群能正常工作
2.当出现两个节点故障时，集群不能正常工作，NN节点进程反复退出，重启。






