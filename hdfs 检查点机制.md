
hdfs checkpoint 机制对于 namenode 元数据的保护至关重要, 是否正常完成检查点是评估 hdfs 集群健康度和[[长期未完成 checkpoint 导致重启 namenode 慢问题|风险]]的重要指标


- editslog : 对hdfs操作的事务记录，类似于WAL，edit log文件以 edits_ 开头，后面跟一个txid范围段，并且多个edit log之间首尾相连,正在使用的 edit log 名字为 edits_inprogress_txid（dfs.namenode.edits.dir）
- fsimage：文件系统的元数据，snn会定时合并将内存中的元数据落盘生成新的fsimage，默认会保存两个fsimage文件，文件格式为fsimage_txid（dfs.namenode.name.dir）
- seen_txid：记录了最后一次 checkpoint 或者 edit 回滚（将 edits_inprogress_xxx 文件回滚成一个新的 Edits 文件）之后的 transaction ID。主要用来检查 NameNode 启动过程中 Edits 文件是否有丢失的情况

![[Pasted image 20221207103015.png]]

检查点将从旧的 fsimage 和编辑日志进行合并,创建一个新的 fsimage

![[Pasted image 20221128143125.png]]


checkpoint 触发由三个参数控制
```
dfs.namenode.checkpoint.period
dfs.namenode.checkpoint.txns
dfs.namenode.checkpoint.check.period

```




### HA 集群的 checkpoint 过程

![[Pasted image 20221128160510.png]]

这里 standby namenode 称为 SBNN，active namenode 称为 ANN
1. SBNN 查看是否满足创建检查点的条件（距离上次 checkpoint 的时间间隔大于等于 dfs.namenode.checkpoint.period
edits log 中的事务条数达到 dfs.namenode.checkpoint.txns 限制）
2. SBNN 将内存中当前的状态保存成一个新的文件，命名为fsimage.ckpt_txid。其中 txid 是最后一个 edit log 中的最后一条事务的 ID（transaction ID，不包括 inprogress）。然后为该 fsimage 文件创建一个MD5 文件，并将 fsimage 文件重命名为 fsimage_txid。
3. SBNN 向 ANN 发送一条 HTTP GET 请求。请求中包含了 SBNN 的域名，端口以及新 fsimage 的 txid。
ANN 收到请求后，用获取到的信息反过来向 SBNN 再发送一条 HTTP GET 请求，获取新的 fsimage 文件。这个新的 fsimage 文件传输到 ANN 上后，也是先命名为 fsimage.ckpt_txid，并为它创建一个 MD5 文件。然后再改名为 fsimage_txid，checkpoint过程完成。



