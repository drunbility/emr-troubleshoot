
加快hdfs datanode节点下线相关参数参考 相关参数解释：
1. dfs.namenode.replication.max-streams：单个DataNode最大同时恢复的块数量，可以间接控制DataNode恢复数据块的带来的网络等压力。
   需要与 dfs.namenode.replication.work.multiplier.per.iteration配置项配合使用。 
2. dfs.namenode.replication.max-streams-hard-limit：最高优先级复制流的数量的硬限制。
3. dfs.namenode.replication.work.multiplier.per.iteration：决定了可以从很多under replication blocks中选出多少个block准备进行复制。如果该参数配置得太小，则dfs.namenode.replication.max-streams配置得再大没有用；可以选出的block数与集群live 的datadnode 成正比。 
4. dfs.datanode.balance.max.concurrent.moves：单个datanode允许同时移动最大块数
5. dfs.datanode.balance.bandwidthPerSec：DataNode用于Balancer的带宽 ，决定了可以从很多under replication blocks中选出多少个block 准备进行复制。如果该参数配置得太小，则dfs.namenode.replication.max-streams配置得再大没有用；可以选出的block数与集群live 的datadnode 成正比。  


参数配置： 在emr控制台hdfs-site.xml中，新增如下配置参数。 
dfs.namenode.replication.max-streams 默认：2 建议： 20 
dfs.namenode.replication.max-streams-hard-limit 默认： 4 建议：40 dfs.namenode.replication.work.multiplier.per.iteration 默认： 2 建议：10 


dfs.datanode.balance.max.concurrent.moves 默认：5 建议： 256 dfs.datanode.balance.bandwidthPerSec 默认：10485760 建议：524288000  
根据集群压力，选择适当的值进行调整，如果调整了前三个参数，则需要重启NN。如果修改了后两个参数，需要重启DN. 在 3.x 版本,可以[[动态更新 hdfs 配置]]

bandwidthPerSec 也可以通过命令直接设置:
`hdfs dfsadmin -setBalancerBandwidth 20971520`  设置了balancer限流20M，该值可以根据实际情况动态设置，建议不超过100M，以免影响业务


设置好参数后,可以[[启动 balancer 脚本]]
