### 现象
背景是用户不规范操作,在 master 节点上启动了很多个 hbase shell ,导致节点进入了僵死状态. 此时 active nn  变成 了 standby 状态,但是 stanby nn 并没有变成 active ,两个 nn 都是 standby 状态

### 排查过程

首先使用 stress 工具来模拟 master 节点进入僵死状态 

`stress  --vm 55 --vm-bytes 1G --vm-hang 100000 --timeout 100000`

执行几次发现, 确实会出现两个 standby nn 的现象.

这种情况是由于  active nn  持有了 zk 选举锁,但是 active nn 处于 hang 死状态了,也不能正常释放 zk 选举锁,zk 选举锁没有释放, stanby nn 就无法抢到锁,也就不能变成 active nn 状态.
刚好此时 health monitor 走到了健康检查周期,无法得到 active nn 的状态.于是 active nn 也变成了 standby 状态.

![[企业微信截图_b8c0b89a-c43c-4a12-b401-8a7238fde1b1.png]]

![[企业微信截图_ab9a4163-2ea5-4017-a8cd-1f8044752d4f.png]]



### 结论

可以配置 zk 锁的超时时间

`ha.zookeeper.session-timeout.ms`   这个值默认是 180000  ,这个值可以配小,但是可能会导致频繁主从切换,

根本原因还是要避免不规范操作, master 节点是重要的节点,机器层面应该避免这个节点出现 Hang 死状态


