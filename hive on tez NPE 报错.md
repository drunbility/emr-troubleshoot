
#### 现象

用户反馈 hive on tez 任务偶现失败,重跑一次又成功了

#### 排查过程

首先拿到报错任务的  yarn 日志,查看日志,发现失败是由于发生了 NPE 错误

```
Caused by: java.lang.NullPointerException

at org.apache.hadoop.hive.ql.exec.persistence.PTFRowContainer.first(PTFRowContainer.java:115)

at org.apache.hadoop.hive.ql.exec.PTFPartition.iterator(PTFPartition.java:114)

at org.apache.hadoop.hive.ql.udf.ptf.BasePartitionEvaluator.getPartitionAgg(BasePartitionEvaluator.java:200)

at org.apache.hadoop.hive.ql.udf.ptf.WindowingTableFunction.evaluateFunctionOnPartition(WindowingTableFunction.java:155)

at org.apache.hadoop.hive.ql.udf.ptf.WindowingTableFunction.iterator(WindowingTableFunction.java:538)

at org.apache.hadoop.hive.ql.exec.PTFOperator$PTFInvocation.finishPartition(PTFOperator.java:349)

at org.apache.hadoop.hive.ql.exec.PTFOperator.process(PTFOperator.java:123)

at org.apache.hadoop.hive.ql.exec.Operator.forward(Operator.java:897)

at org.apache.hadoop.hive.ql.exec.SelectOperator.process(SelectOperator.java:95)

at org.apache.hadoop.hive.ql.exec.tez.ReduceRecordSource$GroupIterator.next(ReduceRecordSource.java:356)

... 17 more

```

根据这个关键词[[Hive jira 搜索小技巧|搜索  hive jira]] ,发现相关 issue 记录

[[HIVE-18786] NPE in Hive windowing functions - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/HIVE-18786)

报错信息一致,找到问题原因


#### 解决办法

issue 里面提到这个是由于 tez container reuse 导致的,那么把 reuse 关掉即可
set tez.am.container.reuse.enabled=false

---
[Solved: NullPointerException (but not always) in GroupBy i... - Cloudera Community - 239765](https://community.cloudera.com/t5/Support-Questions/NullPointerException-but-not-always-in-GroupBy-in-Hive-with/td-p/239765)







