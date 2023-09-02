
yarn的nodemanager进程一直拉不起来，查看日志看到 加载 /sys/fs/cgroup/memory失败:

	Unexpected: Cannot create yarn cgroup Subsystem:memory Mount points:/proc/mounts User:hadoop Path:/sys/fs/cgroup/memory/hadoop-yarn

解决办法:  
在该节点重新执行下以下命令  
sudo mkdir -p /sys/fs/cgroup/memory/hadoop-yarn  
sudo chown -R hadoop:hadoop /sys/fs/cgroup/memory/hadoop-yarn  
sudo ls -al /sys/fs/cgroup/memory/hadoop-yarn


这个目录在机器重启后会消失,所以每次需要手动恢复


>Cgroup是control groups（控制群组）的缩写，是linux系统对单个或者多个进程进行资源管理的一种机制，通过Cgroup，可以实现对cpu、内存、磁盘IO（IO调度权重、iops/bps的限制）、网络（基于tc）的使用限制
>



