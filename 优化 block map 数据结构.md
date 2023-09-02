
[[namenode 内存概要 | hdfs 内存]] 中有一个非常重要的数据结构  blocksmap  .BlocksMap在NameNode内存空间占据很大比例，由BlockManager统一管理，相比Namespace，BlockManager管理的这部分数据要复杂的多
BlocksMap 底层通过 LightWeightGSet 实现，本质是一个链式解决冲突的哈希表。

在 [HDFS-9260](https://issues.apache.org/jira/browse/HDFS-9260) 系列优化中, 也就是 3.x 版本后, BlocksMap 变更为 FoldedTreeSet 实现.接着根据社区的反馈, 这个变更会导致 namenode 以及 [[datanode 持锁导致 hive 任务慢问题|datanode 较大性能问题]],之后还原为 LightWeightGSet 实现:

[[HDFS-13671] Namenode deletes large dir slowly caused by FoldedTreeSet#removeAndGet - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/HDFS-13671)
[[HDFS-15131] FoldedTreeSet appears to degrade over time - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/HDFS-15131)
[[HDFS-15140] Replace FoldedTreeSet in Datanode with SortedSet or TreeMap - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/HDFS-15140)
[[HDFS-16043] Add markedDeleteBlockScrubberThread to delete blocks asynchronously - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/HDFS-16043)







