balancer 是一个单独的进程,可以选择一台空闲的节点运行,注意要使用 hadoop 超级用户来运行

```
su hadoop 
/usr/local/service/hadoop/sbin/start-balancer.sh -threshold 5 -idleiterations 1000
```

参数说明:
```

-threshold 10%(默认)，该参数表明集群磁盘空间达到均衡的条件，越小则说明集群越均衡，同时也耗时越久，建议根据实际情况进行配置
-policy 策略取值: datanode, blockpool。默认是基于datanode进行均衡。blockpool只能在Federation架构下有效，balancer级别为blockpool表示每个datanode上每个blockpool的blocks个数分布均衡。建议保持默认。
-exclude -f <filename>:排除某些节点参与数据均衡操作。
-include -f <filename>:指定某些节点参与数据均衡操作。
-idleiterations : 快移动失败后重试次数,在繁忙集群中可能会出现快移动失败

```



在启动命令中,也可以一并带上 [[hdfs balancer 加速|balancer 加速配置]]

```
hdfs balancer -Ddfs.balancer.movedWinWidth=5400000 -Ddfs.balancer.moverThreads=1000 -Ddfs.balancer.dispatcherThreads=200 -Ddfs.datanode.balance.bandwidthPerSec=100000000 -Ddfs.balancer.max-size-to-move=10737418240 -threshold 20 1>/tmp/balancer-out.log 2>/tmp/balancer-debug.log

```

