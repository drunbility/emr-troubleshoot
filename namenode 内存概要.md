NameNode管理着整个HDFS文件系统的元数据。从架构设计上看，元数据大致分成两个层次：
1. Namespace管理层，负责管理文件系统中的树状目录结构以及文件与数据块的映射关系；
2. 块管理层，负责管理文件系统中文件的物理块与实际存储位置的映射关系BlocksMap，如图1所示[1]。

Namespace管理的元数据除内存常驻外，也会周期Flush到持久化设备上FsImage文件；BlocksMap元数据只在内存中存在；当NameNode发生重启，首先从持久化设备中读取FsImage构建Namespace，之后根据DataNode的汇报信息重新构造BlocksMap。这两部分数据结构是占据了NameNode大部分JVM Heap空间。


![[Pasted image 20221208170719.png]]

除了对文件系统本身元数据的管理之外，NameNode还需要维护整个集群的机架及DataNode的信息、Lease管理以及集中式缓存引入的缓存管理等等。这几部分数据结构空间占用相对固定，且占用较小。

测试数据显示，Namespace目录和文件总量到2亿，数据块总量到3亿后，[[hdfs 所需内存预估|常驻内存]]使用量超过90GB。



内存全景
#todo 

