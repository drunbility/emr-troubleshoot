### 现象
用户反馈重启 hs2 后,启动过程非常长,要等8分钟后, hs2 才能完全启动完成,对外提供端口来连接

### 排查过程

采集进程堆栈,结合 hs2 的日志,发现 hs2 的进程是卡在 hive 请求 hms 的 thrift  rpc 方法里面,

`org.apache.hadoop.hive.ql.metadata.Hive#getAllTableObjects`

按照 [[hive 执行 desc table 较慢问题#^92768d]]  中比较 git 分支方法,在 3.1.3 分支中找到一个提交记录:

[[HIVE-15436] Enhancing metastore APIs to retrieve only materialized views - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/HIVE-15436)


合入这个 patch ,问题解决




