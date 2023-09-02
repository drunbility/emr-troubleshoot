
hdfs 在启动的过程中需要做全量块汇报，此时会执行du -sk命令（去机器上ps -ef ｜ grep du），如果小文件多或者磁盘有问题，会导致卡在这一步

启动后也会定期（默认10分钟）使用 du -sk 命令统计BP的大小，在大硬盘机器上该操作耗时将会很长（可能超过10分钟）。这会导致 iowait 以及 load的提升。

![[Pasted image 20221123154930.png]]


### 2.8.0 后版本

社区为了解决这个问题，主要在两个方面进行了改造, 
https://issues.apache.org/jira/browse/HADOOP-9884

使用 df 命令替换 du
`fs.getspaceused.classname : org.apache.hadoop.fs.DFCachingGetSpaceUsed`

允许用户自定义检查间隔时间
```
/**  
 * @see  
 * <a href="{@docRoot}/../hadoop-project-dist/hadoop-common/core-default.xml">  
 * core-default.xml</a>  
 */  
public static final String  FS_DU_INTERVAL_KEY = "fs.du.interval";  
/** Default value for FS_DU_INTERVAL_KEY */  
public static final long    FS_DU_INTERVAL_DEFAULT = 600000;
```


### 2.8.0 前版本

1. 使用 df 命令替换 du
![[Pasted image 20221123172225.png]]

**注意:**
du命令对统计的文件逐个进行fstat系统调用，获取文件大小。它的数据是基于文件获取，可以跨多个分区操作。df命令使用statfs系统调用，直接读取分区的超级块信息获取分区使用情况。它的数据基于分区元数据，只能针对整个分区。

删除大量的文件后，du命令不会在文件系统目录中统计删除的文件。如果此时还有运行中的进程持有被删除文件的句柄，那么此[[shell 常用查找删除命令#^6396f2|文件就不会真正的在磁盘中被删除]]，分区超级块中的信息也就不会更改，df命令仍会统计被删除文件的信息。导致 du 和 df 结果不一致



2. 修改/data1/qcloud/data/hdfs/current/BP-1584531057-10.134.133.40-1636615787348/current/dfsUsed，第一个值是使用的空间，第二个是时间戳，如果时间错与当前的时间满足差值（dfs.datanode.cached-dfsused.check.interval.ms），则会直接读取cache的值

![[Pasted image 20221123172337.png]]

---

补充问题:
由于磁盘文件系统 inode 的差异导致 du -sk 命令直接卡死的分析[[du -sk 命令卡死问题]]





