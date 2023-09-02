
当 journalnode 异常,导致 namenode 重启回放 edit logs 失败的时候,可以使用  namenode 的 recovery  启动模式,来恢复部分 edits logs 数据

使用命令 hdfs namenode -recover

当处于恢复模式时，namenode 将在命令行以交互方式提示可以采取哪些行动来恢复数据。

如果要跳过提示，可以指定 -force 选项。此选项将强制恢复模式始终选择第一个选项。

Recovery 模式可能会导致丢失数据



对于一些对于 SLA 要求特别高的用户, recovery 模式恢复存在风险,此时只能在源码级别修改 edit log 回放逻辑来跳过错误了:

*处理了一些空指针异常、发生冲突的时候会强制用editlog的block ID覆盖fsimage block ID，而不是抛异常*





