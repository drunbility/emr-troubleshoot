
datanode 发生坏盘,用户更换盘后,在对新盘做格式化的时候,为了节省格式化时间,在默认的格盘命令中加入了参数:
1. mkfs.ext4 -N 4291584 /dev/vdb   
2. mkfs.ext4  /dev/vdb

第一种方式 指定了 inode 数量  ,格盘时间只需39 s 
第二种方式是默认方式,格盘时间需要 18m

两种方式导致的最大差异是在 ext4  文件系统中每个 group 内 inode 数量的巨大差异:
48 inodes per group  VS 8192 inodes per group 
![[Pasted image 20221123200849.png]]


du 命令读取的是文件或目录的inode信息，根据ionde信息提取出文件的大小等信息进行统计

在这个 hdfs 的 data dir 目录下,大概有 85 w 个文件 . 这些文件在写入磁盘的时候,每个group 设置的inode数量不够，造成跨group分配。如果大量跨group 分配 inode, 造成inode 在硬盘上物理位置是跳跃的，引起du 操作的读请求无法有效合并IO，从而引起导致时间大幅加长甚至卡死


### 解决办法
1. [[du命令优化]]
2. 重新格式化问题环境的相应数据盘，例如格式化为默认inode 数量

---

[[inode 不足导致 hdfs 无法写入问题]] 







