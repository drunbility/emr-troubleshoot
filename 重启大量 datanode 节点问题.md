
对于BlockReport类型的RPC请求，重启全集群DataNode与重启NameNode，RPC处理时间有较大差别

```
if (storageInfo.getBlockReportCount() == 0) {  
  // The first block report can be processed a lot more efficiently than  
  // ordinary block reports.  This shortens restart times.  blockLog.info("BLOCK* processReport 0x{}: Processing first "  
          + "storage report for {} from datanode {}",  
      strBlockReportId,  
      storageInfo.getStorageID(),  
      nodeID.getDatanodeUuid());  
  processFirstBlockReport(storageInfo, newReport);  
} else {  
  invalidatedBlocks = processReport(storageInfo, newReport, context);  
}
```

可以看到NameNode对BlockReport的处理方式仅区别于是否为初次BlockReport。初次BlockReport显然只发生在NameNode重启期间。

processFirstBlockReport：对Standby节点（NameNode重启期间均为Standby），如果汇报的数据块相关元数据还没有加载，会将报告的块信息暂存队列，当Standby节点完成加载相关元数据后，再处理该消息队列； 对第一次块汇报的处理比较特别，为提高处理效率，仅验证块是否损坏，然后判断块状态是否为FINALIZED状态，如果是建立块与DN节点的映射，其他信息一概暂不处理。

processReport：对于非初次块汇报，处理逻辑要复杂很多；对报告的每个块信息，不仅会建立块与DN的映射，还会检查是否损坏，是否无效，是否需要删除，是否为UC状态等等。


当短时间重启大量DataNode时，由于处理BlockReport逻辑不同,可能会造成 namenode rpc 队列堆积大量 BlockReport 请求,从而是 namenode 持续无法响应正常的读写请求


### 解决办法
1.  减小 dfs.blockreport.split.threshold ,将 block report  切分成每个盘来汇报
2. 采取滚动方式，每次重启一台,中间间隔1到3分钟
3. 如果队列堆积严重,可以手动清理队列 dfsadmin -refreshCallQueue
4. 可以在重启 datanode 后再重启 namenode ,使得namenode 进入初次 block report 处理逻辑

