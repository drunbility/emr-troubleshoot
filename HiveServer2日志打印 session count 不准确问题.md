### 现象
用户反馈 HiveServer2 打开的连接数监控指标和节点上实际的 tcp 链接数对不好

![[企业微信截图_12370eac-52f7-4553-91ea-d95951c4530f.png]]

![[企业微信截图_a7798e65-80c1-4632-a8b6-e59d56981d56.png]]


### 排查过程

分析源码,首先看到这个指标的含义,指标的定义在类 

`org.apache.hadoop.hive.common.metrics.common.MetricsConstant` 中
找到这个指标定义 `org.apache.hadoop.hive.common.metrics.common.MetricsConstant#OPEN_CONNECTIONS`

接着看这个指标是如果计算的,通过查找找个这个变量的 usage 位置,发现这个 connections 
的计算在一个 [[thrift 基本概念|thrift server handler]] 中 `org.apache.thrift.server.TServer#setServerEventHandler`

![[Pasted image 20230401212008.png]]

这里当时并没有继续排查下去,只是跟用户解释这个指标代表的是 thrift server 中 context 上下文的数量,并不是直接代表着节点上  tcp 的连接数.
这里肯定是有 tcp 连接的释放问题,没有走到 delete 监听方法内,当时判断该指标并不影响业务,就没有继续追查下去.

直到后来用户又提出了同样的问题,现象是一样的,连接数飙高,但机器上实际连接并没有,这次问题比较严重
![[企业微信截图_16781571142345.png]]

连接数达到了2000, 这个时段用户反馈客户端无法连接 HiveServer2 .

这个 2000 跟  hs2 thrift server 的 [[Hive 中的几个线程数配置项|处理线程]] 上限 `hive.server2.thrift.max.worker.threads`  配置的值一样,并且图上有一个持续2000 的平峰,这种情况就是典型达到了某个资源的上限,导致的平峰等待,期间客户端无法连接.

但根据当时机器的监控来看,当时对这台 hs2 机器上的 tcp 连接数并不高.
初步断定是由于连接数没有正常关闭,导致 worker thread 也没有释放,累计到上限,从而发生客户端无法连接了.

回到上上图中 deleteContext 方法,资源没有释放,肯定就是没有走到 deleteContext 方法里面.为什么不会走到这个方法里面,可以看到这里是一个 thrift 的 tcp 事件 handler ,没走到这里,就是没有接收到对应事件,那么是不是由于客户端没有发送对应事件?


- 在 hive jira 中搜索关键词  `connection close`  ,指定 Resolution 为 Fixed  ^e31c12

![[Pasted image 20230401220308.png]]

逐个翻阅,果然找到一个描述非常吻合的 case :
[[HIVE-24694] Early connection close to release server resources during creating - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/HIVE-24694)


### 结论
查看 issue 类修复代码: [HIVE-24694: Early connection close to release server resources during creating by dengzhhu653 · Pull Request #1920 · apache/hive (github.com)](https://github.com/apache/hive/pull/1920/files)

果然是 jdbc 客户端的 close 逻辑不完备,合入这部分代码到客户端即可修复


---
相关 jira:
[[HIVE-14296] Session count is not decremented when HS2 clients do not shutdown cleanly. - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/HIVE-14296)


