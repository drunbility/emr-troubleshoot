VERSION文件主要保存集群版本相关的信息,总共有三种 version 文件:

1. namenode的 version, 位于位于/data/emr/hdfs/namenode/current/VERSION
2. datanode 的外层 version ,位于/data/qcloud/data/hdfs/current/VERSION，每个盘下面都会有
3. datanode 的内层 version,位于 /data/qcloud/data/hdfs/current/BP-873099873-10.0.0.93-1637761694672/current

datanode 容易出现VERSION文件的问题，一般都是手动干了啥
1. datanode的VERSION文件丢失，导致dn启动失败
   - 如果是内层的VERSION文件丢失，则直接从本机的相同的blockpool下拷贝到对应位置即可
   - 如果是外层的VERSION文件，从其他机器的外层拷得VERSION文件到本目录下，并修改storageID和datanodeUuid

2. datanode的多个数据目录下的VERSION文件中datanodeUUID不一致
   改为一致的，并且保证与其他节点的不重复（可以按生产UUID的格式统一修改下）

3. 多个datanode的VERSION文件中datanodeUUID一样 
   此时只有一个datanode能注册上去，需要排查nn的日志发现问题，修改掉其他节点的datanodeUUID







