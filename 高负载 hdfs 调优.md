

###  减少 datanode 增量块汇报  
当 datanode 上新写完一个块，默认会立即汇报给 namenode。在一个大规模 Hadoop 集群上，每时每刻都在写数据，datanode 上随时都会有写完数据块然后汇报给 namenode 的情况。因此 namenode 会频繁处理 datanode 这种快汇报请求，会频繁地持有锁，其实非常影响其他 rpc 的处理和响应时间。通过延迟快汇报配置可以减少 datanode 写完块后的块汇报次数，提高namenode 处理 rpc 的响应时间和处理速度. 

```
<property> 
<name>dfs.blockreport.incremental.intervalMsec</name>
<value>300</value>  
</property>    
```

### 开启service rpc
在规模较小时没必要，但是不影响使用，但是规模较大时，QPS很高情况下非常有必要
dfs.namenode.servicerpc-address.HDFS82235.nn1 master1:port
dfs.namenode.servicerpc-address.HDFS82235.nn2 master2:port
dfs.namenode.service.handler.count  32  (dfs.namenode.handler.count 未开启时是这个配置)
注：上述的集群名称，节点IP，端口都要根据实际情况动态调整
> [namenode 性能优化 RPC队列拆分 - 北漂-boy - 博客园 (cnblogs.com)](https://www.cnblogs.com/yjt1993/p/11198855.html)


### namenode [[jvm 启动参数]]调优


### 通用参数调整
1. 增加写QJM超时时间
这个参数默认配置比较小会导致NN退出，导致文件系统切换甚至主节点不可用
dfs.qjournal.write-txns.timeout.ms 40000
2. 检查点事务数量限制数
默认每小时或者达到一百万时进行检查点，在写量较大的大集群中，会导致频繁的检查点操作，加重NN压力
dfs.namenode.checkpoint.txns 5000000
3. DN心跳延迟，增大客户端重试次数：
dfs.client.block.write.locateFollowingBlock.retries 5->10

### 开启本地读
本地读有利于减少DN的压力，减少读操作产生的网络流量
dfs.client.read.shortcircuit  true
dfs.domain.socket.path  /var/lib/hadoop/dn_socket

### [[du命令优化]]

---
### 注意事项
修改上述参数，需要 [[重启 hdfs 规范步骤]]，开启本地读，还需要重启hdfs客户端
开启service rpc需要额外做以下操作（注：hdfs分钟级不可用）：
1）控制台停止ZKFC
2）在节点上用hadoop用户执行
hdfs zkfc -formatZK  
也可以手动删除 zk 上的znode : rmr /hadoop-ha/HDFS82516/ActiveBreadCrumb
3）控制台启动ZKFC




