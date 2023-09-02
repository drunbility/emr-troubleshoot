
hdfs 在启动流程需要加载 edit logs,如果 edit logs 有错误, hdfs 将会无法上线

比较常见的原因是:
误删editslog、JournalNode节点有断电、数据目录磁盘占满、网络异常时

常见报错如下:
```
java.io.IOException: Gap in transactions. Expected to be able to read up until at least txid 813248390 but unable to find any edit logs containing txid 363417469
```

可以[[动态开启DEBUG 日志级别]]定位报错位置

解决办法：
1. 查看其它的JournalNode的数据目录或NameNode数据目录中，有没有连续的该序号相关的连续的edits文件。如果可以找到，复制一个连续的片段到该JournalNode
2. 使用[[ namenode recovery 模式]]  跳过 edits 错误 
3. 使用 [[edits viewer 修复错误 edit 文件]]
4. [[从 active nn 恢复 standy nn]]
5. 以上如不能解决,只能[[从fsimage恢复namenode]]  来上线 namenode


---

高频故障 issue:
https://issues.apache.org/jira/browse/HDFS-15175





