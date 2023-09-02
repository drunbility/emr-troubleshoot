为了解决出现两个 active nn 而导致元数据的一致性问题

QJM(Quorum Journal Manager) 有非常重要的一点,它同时只允许一个 namenode 去写入 JournalNodes,这样才能保证hdfs 元数据的正确性.


但是在故障转移发生的场景下,有可能出现两个 active namenode ,此时旧的 active namenode 仍然在向客户端提供读请求,并返回给客户端过时的请求结果.

zkfc 层面本身有这样的防脑裂机制,在切换中，zkfc 会调用a nn的rpc接口使其成为standby,如果rpc调用失败（NNGC 超时），则会跟据配置的fence信息进行强制策略隔离，一般是杀掉主NN

有两种配置：
1. sshfence
SSH 到 Active NameNode 并终止进程

配置如下：
 dfs.ha.fencing.methods sshfence(root:22)
 dfs.ha.fencing.ssh.private-key-files 无密钥的私钥文件
 其中括号里面是用户名+ssh端口

2. shellfence
执行该配置里的shell脚本进行后续操作
dfs.ha.fencing.methods shell(/bin/bash a.sh)
会执行用户自定义的a.sh，你可以在a.sh中自己实现杀掉老NN的逻辑，更加灵活方便

两种策略可以同时配置，且都可以配置多个例如
 
```
 <property>
	<name>dfs.ha.fencing.methods</name>
		<value>
			sshfence(root:22)
			shell(/bin/bash a.sh)
			shell(/bin/bash b.sh)
		</value>
 </property>
```
 fence会依次执行该配置的函数，直到某个fence成功为止










