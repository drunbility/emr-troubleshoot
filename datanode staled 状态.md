
默认情况下,datanode 在失去心跳 30 s 后,就会被 namenode 标记为 stale 的状态
如果接着失去心跳, 在 10.5 分钟后,就会被标记为 dead 状态了.
计算公式为
`dfs.namenode.stale.datanode.interval < last contact < (2 * dfs.namenode.heartbeat.recheck-interval)`


相关的参数如下

-   dfs.heartbeat.interval - default: 3 seconds
-   dfs.namenode.stale.datanode.interval - default: 30 seconds
-   dfs.namenode.heartbeat.recheck-interval - default: 5 minutes
-   dfs.namenode.avoid.read.stale.datanode - 2.x 版本默认为 false ,应该设置为 true ,避免对 stale 节点读写
-   dfs.namenode.avoid.write.stale.datanode - 2.x 版本默认为 false ,应该设置为 true ,避免对 stale 节点读写


---
[Solved: How to identify stale datanode? - Cloudera Community - 96336](https://community.cloudera.com/t5/Support-Questions/How-to-identify-stale-datanode/td-p/96336)







