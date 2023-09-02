
## cloudera 官方粗略算法
### namenode
在做集群规划的时候,需要规划 master 节点,进而需要评估 namenode ,datanode jvm heapsize 所需内存

一般来说,每百万个 block 需要 1gb jvm 内存,这种计算方式综合考虑了 namespace 对象,其他必要的数据结构以及 RPC 服务等负载.这种预估出来的内存一般比实际占用要充足


如果要详细计算 namespace 的 内存大小,就要知道 namespace 里面的对象数,也就是文件(inode) 数和 副本(block)数之和.然后每一个对象大概占用 150 bytes.
比如一个 192M 的文件,block size 是128 M,那么有2个 block
所以所需内存是  (1+2) * 150 = 450 bytes
注意,文件的副本数不会影响 namespace 的内存占用 ,副本数只会影响磁盘占用,所以上面计算的 block 数量是2
这里只计算了 namespace 对象占用,没有考虑其他工作负载(其他对象以及RPC 调用)


### datanode

datanode 一般最低配置 4 gb 的内存,然后每超过 4百万个副本，每百万个副本增加 1 GB 的内存。例如，5百万个副本需要 5 GB 的内存。


## 社区精确算法

1. NameNode内存使用量预估模型：Total=218 * num(INode) + 168 * num(Blocks)；
2. 受JVM可管理内存上限等物理因素，180G内存下，NameNode服务上限的元数据量约700M。

---
[Hardware Requirements | 6.x | Cloudera Documentation](https://docs.cloudera.com/documentation/enterprise/6/release-notes/topics/rg_hardware_requirements.html#concept_fzz_dq4_gbb)
[NameNode内存详解 - Hexiaoqiao](https://hexiaoqiao.github.io/blog/2016/07/21/namenode-memory-detail/)






