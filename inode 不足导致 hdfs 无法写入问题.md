
当磁盘的 inode 不足的时候, hdfs 会抛出无法写入磁盘的异常,即使当前磁盘可用存储空间是足够的


![[企业微信截图_16641609893397.png]]

![[企业微信截图_102dd760-9b11-4811-9391-c8cd7845881b.png]]

![[wecom-temp-10410-b29da71da90ee504632e8780c1ff2850.png]]



文件在存储到磁盘中的时候，会同时用到inode和block，inode保存文件属性信息，包括文件名，大小，权限，时间，存储位置等，而block中则保存实际的数据，

所以如果inode用完的话，即使磁盘还有空间也无法创建新文件了。

发生这种情况的原因一般是系统中小文件过多，占用了大量的inode节点，所以需要清理 hdfs / hbase 文件,或者[[shell 常用查找删除命令|查找删除磁盘上其他文件]]



---

[理解inode - 阮一峰的网络日志 (ruanyifeng.com)](https://www.ruanyifeng.com/blog/2011/12/inode.html)


