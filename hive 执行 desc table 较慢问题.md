#### 现象
用户反馈,他们执行一个简单的 desc table 查询,返回比较慢,需要 5s 才能返回结果


#### 分析

发现 desc 其他表也有这种现象.这些表有一个共同特点,就是分区数比较多,上万个分区.
怀疑这么慢是由于对分区进行了遍历操作导致的.

#### 排查

要验证上面的猜想,最直接的方法就是去看代码,首先找到这部分的代码位置.
要找到具体的代码位置,有两种方法
- 一种是通过[Hive远程debug](https://blog.csdn.net/qq_39002724/article/details/114644803) 断点找到具体的类和方法
- 还一种是通过 arthas 抓取 desc table 的火焰图,找到详细的调用栈
一般使用第二种方法较块.最终找到具体的方法位置

`org.apache.hadoop.hive.ql.exec.DDLTask#describeTable`

看到里面确实做了所有分区的遍历

![[Pasted image 20230216210300.png]]

可以看到上面构造了迭代器,并对所以分区进行了 for each 遍历,所以能够确定是这块的遍历逻辑导致了查询耗时较久.


#### 修复方法

为什么 desc table 要去遍历所有的分区,这里是有问题的.点开 for each 行 git blame ,看到这个 commit 信息

![[Pasted image 20230217144211.png]]

查看 jira 地址 [[HIVE-16098] Describe table doesn't show stats for partitioned tables - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/HIVE-16098)
查看 jira 的 pr 信息,可以看到这个 pr 的说明:
目前 desc table 语句中指定了分区,就列出分区的统计信息,这个 pr 以后,如果没有指定分区,那就会列出所有分区的聚合信息

	hive 2 旧版本 isssue 快速修复

^92768d

这里看上去是没必要的.对于 hive ,这种情况可以去比对新版本的代码,看新版本中这块有没有优化
直接在 idea 该类中点击右键 -> Git -> Compare With Branch ,然后选择下一个版本 3.1.3 
查看对比,同时右键打开 3.1.3 分支的 Git Blame 信息,看到这里有一个明显的差异

![[Pasted image 20230217152721.png]]

点开 3.1.3 这个 git blame 信息,看到提交信息 `HIVE-24964: Backport HIVE-22453 to branch-3.1 `
查看 jira 信息 [[HIVE-24964] Backport HIVE-22453: Describe table unnecessarily fetches partitions - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/HIVE-24964)

果然这块优化了这块遍历逻辑, 只有`describe extended/formatted table`  才会去聚合分区统计信息

```
#### Description

When users have highly partitioned table (10k+), all describe commands will take a very long time to complete. This change will make it such that only "describe extended/formatted table" will start gathering partition statistics. "Describe table" will not collect the stats.
```






