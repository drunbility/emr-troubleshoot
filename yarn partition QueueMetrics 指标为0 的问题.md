#### 问题描述

用户反馈采集 yarn jmx 指标  `curl localhost:19984/metrics |grep Hadoop_ResourceManager_AllocatedVCores`  发现 QueueMetrics 队列 AllocatedVCores 指标都为0 ,但实际上在 yarn ui 上看,这些队列都是正常运行了任务的. 

接着在 yarn web ui 查看排查 schduler [[yarn partition|partition]],发现 default partition 下面的一些 metrics 指标都为0 

![[wecom-temp-79297-2aaff65be7ccd9cec13da6ab8fc0d238.png]]

当时怀疑是社区一点某些 isssue 造成的


#### 结论
后来定位到这不是由于 bug 引起,而是版本不支持问题
在用户 的 yarn  3.2.1 版本 RM  上 的 QueueMetrics  是不支持 partition 的,也就是只能去  default partition 的指标

PartitionQueueMetrics 这是一个子类,是在 3.2.2 版本新增,也就是能够采集特定分区下某一个队列的metrics 指标

相关 issue :[[YARN-6492] Generate queue metrics for each partition - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/YARN-6492)














