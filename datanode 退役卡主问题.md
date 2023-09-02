
datanode 节点下线,本质问题是要把这个节点上 block / 副本 转移到其他 datanode 节点上

如果下线时间慢,可以调整副本复制相关的三个参数,见[[hdfs balancer 加速]]

如果长时间未完成,卡住了的状态,就要去排查为什么 block 为什么不能完成搬运:

常见的情况是:
1. 下线 dn 上还有 dfsclient 保持连接,并持续往里面写数据,或者这个 dn 还持有未释放的客户端租约
   
   这种情况可以通过命令   
   `hdfs fsck -openforwrite /`  
   `hdfs dfsadmin -listOpenFiles `   
   `hdfs fsck / -files -blocks -locations | grep Under | grep "Replicas is"`
   来找到这些文件 (==在 3.x 版本可以加上  -blockingDecommission 精确找到阻塞下线 dn 的文件==).
   找到这些文件后,可以通过命令
   `hdfs debug recoverLease -path  ${path}  -retries 3`   手动释放租约
   也可以通过命令来手动驱逐正在写 这个 datanode 的客户端 
   `hdfs dfsadmin -evictWriters <datanode_host:ipc_port> `
2. 除了这个 dn 外,没有其他的 dn  来接纳这些副本了,有两种情况
   -  可用 dn 数低于副本策略的文件副本数
   - 可用 dn 数低于 ec 策略所需的数据快和检验块数     
   这种情况可以通过命令 
   `hdfs fsck / -files -blocks -locations | grep Under | grep "Replicas is"`    
   找到哪些块处于复制中的状态,并且为什么复制无法完成.找到相应的目录后可以通过命令将文件的副本数减少来通过复制
   `hadoop fs -setrep -w -R 3 /user/hadoop/dir1`








