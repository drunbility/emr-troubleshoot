
某线上集群反馈其线上 hive 报表任务突然变慢,导致其下游所有任务产出延迟,并且能够稳定复现.经过运行测试任务,抓取堆栈,最终定位到问题跟 [[优化 block map 数据结构]] 有关

最终结论:
此问题属于开源hadoop的bug，问题单号：HDFS-15131
问题原因是hdsf的IBR使用FoldedTreeSet存储块上报信息，但是由于FoldedTreeSet存在同步锁，随着运行时间推移，datanode的读取性能下降，导致在部分节点上出现长尾的mapTask,最终通过合入关联 patch  HDFS-15140  解决


