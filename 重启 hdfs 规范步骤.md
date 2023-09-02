
[[HDFS HA 背景]] with QJM架构下，NameNode的整个重启过程中始终以SBN（StandbyNameNode）角色完成。与前述流程对应，启动过程分以下几个阶段：
0、加载FSImage；
1、回放EditLog；
2、执行[[hdfs 检查点机制|Checkpoint]]；（非必须步骤，结合实际情况和参数确定，后续详述）
3、收集所有DataNode的注册和数据块汇报后退出安全模式；



### datanode 启动过程

1. 拿到数据存储路径（dfs.datanode.data.dir），检测可用盘（可读、可写、可执行），并返回可用的目录列表，如果没有可用的磁盘，则会异常退出（error：number of data directories should be > 0）
2. 实例化管理磁盘目录的DataStorage，管理与组织磁盘存储目录
3. 启动http和RPC服务
4. datanode向namenode注册和心跳，开始全量块汇报


### 重启hdfs 规范流程
业务低峰期,先重启 standy namenode ,然后控制台操作主备切换,切完后,重启另外一台 standby namenode ,接着[[重启大量 datanode 节点问题|滚动重启 datanode]]







