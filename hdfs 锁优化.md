

### nameode FSNamesystemLock 全局锁优化

社区有一些关于 namenode 拆锁的提案,但都处于开放的状态.

社区有一个专门的标记在跟踪这个目标 Target Version/s :
[Fine-Grained Locking](https://issues.apache.org/jira/browse/HDFS-16125?jql=project%3D%22HDFS%22%20AND%20%22Target%20Version%2Fs%22%3D%22Fine-Grained%20Locking%22%20ORDER%20BY%20priority%20ASC)

已经实现了的是增加对全局锁状态的 jmx 监控,然后在业务上去识别并优化这种情况

[[HDFS-10872] Add MutableRate metrics for FSNamesystemLock operations - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/HDFS-10872)




### datanode 读写锁优化


拆分全局锁:[[HDFS-15382] Split one FsDatasetImpl lock to volume grain locks. - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/HDFS-15382)

配置优化:
- dfs.blockreport.incremental.intervalMsec	300	 IBR延迟批量上报，写单dn的数据量会随着volume增加而加大，增加此项，可减少namenoode rpc请求压力
- ipc.server.listen.queue.size	2048	提升DN backlog配置参数，防止dn因并发连接过多导致SYNC丢包问题
- dfs.blockreport.initialDelay	600	block report推迟上报的加盐参数（random delay取值范围），防止出现dn同时上报blockreport，引起block report storm
- dfs.namenode.full.block.report.lease.length.ms	1800000	block report租约过期时间，此租约功能主要为了解决block report storm而出现。单机block增多，需要的租约时间时间也需要相应增大
- dfs.namenode.replication.max-streams	500	高优复制流上限配置，用于降低单盘故障影响时间
- dfs.namenode.replication.max-streams-hard-limit	1000	整体复制流上限配置，用于降低单盘故障影响时间
- dfs.namenode.replication.work.multiplier.per.iteration	5	立即下发的待复制block数=存活的dn数x此配置值，用于降低单盘故障影响时间


### 优化auditlog

HDFS原生架构中审计日志是使用log4j框架进行输出，但是因为log4j内部很多地方使用了 synchronized 关键字， 当存在大量并发请求调用log4j相应接口打印审计日志时，将会由于竞争 synchronized 影响性能，并且官方也不再对log4j框架进行维护.

log4j2框架是对log4j的版本升级，在日志打印性能上有数倍的提升:


[[HADOOP-12956] Inevitable Log4j2 migration via slf4j - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/HADOOP-12956)





---
[HDFS锁机制优化方向讨论 - Hexiaoqiao](https://hexiaoqiao.github.io/blog/2019/04/26/discussion-on-the-optimization-of-hdfs-global-lock-mechanism/)
[关于Hdfs拆锁的学习和思考 - 简书 (jianshu.com)](https://www.jianshu.com/p/d78c5ab1247c)

