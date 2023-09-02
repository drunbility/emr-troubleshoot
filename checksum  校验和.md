

checksum 校验和有两种,在Hadoop 3.1.1 以前 getFileChecksum 接口返回的是 MD5MD5CRC32，由 Datanode 根据 512 字节 chunk 的 CRC32 计算 MD5 得到块校验值，然后由客户端根据块校验值，计算得到文件校验值。然而，由于不同文件系统的 chunk 和 block size 值可能不一样，因此，该算法实现不适用于跨文件系统之间的数据校验.

为解决跨文件系统的数据校验问题， Hadoop 3.1.1 提供了一个 COMPOSITE_CRC 可选项，用户可通过指定该选项调用 getFileChecksum 接口，此时会返回 CRC32C 校验和，该计算方式和 chunk size 以及 block size 无关，可以作为跨集群和跨文件系统的校验和算法：

```
[hadoop@172 root]$ hadoop fs  -Ddfs.checksum.combine.mode=COMPOSITE_CRC -checksum  /data/test.txt
/data/test.txt  COMPOSITE-CRC32C        6a732798
```


拷贝过程中,容易出现坏块,可以把坏块文件找出来,然后重新进行重新[[拷贝修复坏块文件]]




