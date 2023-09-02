
hdfs 上文件的删除是有 namenode 和 datanode 共同完成的


![[Pasted image 20221222114733.png]]



### namenode端

1. cli提交 删除文件 command；
2. FileSystem会调用具体delete操作;
3. delete操作会由DFSClient通过RPC将delete请求发送给NameNode；
4. nameNode接收请求后，会该操作交由namesystem(名字空间)来处理,该方法会加上全局的写锁；
5. nameSystem会首先校验权限，其次校验namenode状态是否为active。然后从namesystem、blockMap中移除删除的目录或文件，同时将待移除的 block 添加到BlockManager.invalidateBlocks（内部定义数据结构）中；该结构维护某个具体的DataNode与待删除数据块之间映射（Map）；
6. 其中blockManager中的ReplicationMonitor会周期从BlockManager.invalidateBlocks结构中选出若干DataNode（默认选择集群live节点总数的32%）节点，然后从该节点上待删除的数据块列表中选择出一次性最多删除数据块大小（默认1000）的数据块添加到具体的DataNodeDescriptor.invalidateBlocks对象中，同时删除BlockMap.invalidateBlocks对应的数据块；该过程将BlockMap中管理的invalidateBlocks上待删除的数据块分多次的添加对应的DataNodeDescriptor.invalidateBlocks中,待相应的DataNode发送心跳时NameNode会直接通过DataNodeDescriptor中的invalidateBlocks结构中选出本次心跳需要删除的数据块。


### datanode端

1. DataNode周期性向NameNode 发送心跳信息
2. 接收NameNode返回的Command，其中Command分为INVLIDATE(执行删除操作)、TRANSFER（执行数据块移动操作）、REGISTER（执行节点注册）、RECOVERBLOCK（修复block操作）等;
3. 接收返回的Command，DataNode会根据Command类型执行响应的操作，在这里处理删除操作（INVALIDATE），DataNode会掉用BlockPoolSliceScanner执行delete操作，从DataNode的blockMap中删除响应数据块，然后异步删除本地磁盘上的文件，等待下次心跳将这些变更通知NameNode；


###  大目录删除的问题

从上面可以看到, FSNamesystem.delete 方法会加上[[hdfs 锁优化|全局的写锁]],在锁释放之前,namenode 是无法处理其他的 rpc 请求的.所以,如果这个过程执行的过程较长,那么就会造成业务上的一系列的读写异常.比如:
- hdfs client 端报连接超时
- yarn 提交任务失败,因为 yarn 无法上传依赖到 hdfs 上
- datanode 变成 [[datanode staled 状态|stale 状态]],因为 namenode 没及时处理 dn 心跳,进而导致写hdfs  pipeline 失败

优化措施:
1. 分批次删除 block ,减少持锁的时间.比如增加配置 `dfs.namenode.block.deletion.increment`  ,减小每次删除 block 数,减少持锁时间 
   [[HDFS-13831] Make block increment deletion number configurable - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/HDFS-13831)
2. [[优化 block map 数据结构]]
3. 对于大量  pending deletion blocks  ,增加 睡眠时间,此方案还在实现中 [[HDFS-15994] Deletion should sleep some time, when there are too many pending deletion blocks. - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/HDFS-15994)









