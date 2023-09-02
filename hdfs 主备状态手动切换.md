
### 开启自动切换时  ( dfs.ha.automatic-failover.enabled  true)


1. 当active节点正常时，执行hdfs haadmin -transitionToStandby 命令可以将active的namenode节点转换成standby状态，同时原来的standby自动切换为active。
2. 当active节点正常时，使用hdfs haadmin -transitionToActive 命令对两个namenode节点切换都不起作用。
3. 当加了--forceactive 参数，可以将standby的namenode节点转换成active状态，另外的节点自动转为 standby。
4. zk的节点ActiveBreadCrumb, ActiveStandbyElectorLock数据会自动切换。
5. `hdfs haadmin [-failover [--forcefence] [--forceactive] <serviceId> <serviceId>]` 命令在配置故障自动切换（dfs.ha.automatic-failover.enabled=true）之后，无法手动进行

### 关闭自动切换时

1. hdfs haadmin -transitionToStandby将active的namenode节点转换成standby状态, 因此可能存在两个standby的情况。
2. hdfs haadmin -transitionToActive 将standby转为 active。
3. zk对应节点只有ActiveBreadCrumb（遗留未删除的），没有ActiveStandbyElectorLock 节点。











