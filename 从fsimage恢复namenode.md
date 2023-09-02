跳过 edit logs 回放错误,直接从 fsimage (备份)来恢复 namenode

1. 停nn dn jnode zkfc
2. 删除jnode下除了version意外的其他文件
3. 启动jnode，看有没有报错
4. 删除一台namenode 下面的current 除了fsimage fsimage_md5 version以外的其他文件
5. 启动nn 等待fsimage加载完毕，进入安全模式
6. 启动该nn节点的zkfc
7. 手动离开安全模式，看nn是否变成了active
8. 启动datanode，等待块汇报完成
9. hdfs fsck检查下
10. 去第二台nn current下面清除除了VERSION外的其他所有文件，包括fsimage、editlog，然后在这台nn上执行 hdfs namenode -bootstrapStandby
11. 启动第二台nn
12. 启动第二台nn上的zkfc


