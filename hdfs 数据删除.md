### 回收站

启用回收站功能后( fs.trash.interval 设置大于0启用回收站,为零则是禁用)，使用 rm 命令从 HDFS 中删除时，文件或目录不会立即被清除，它们将被移动到回收站 current 目录中`/user/<username>/.Trash`，回收站中的文件和目录通过 mv 还原回来. 在 回收站 currrent 目录中文件在经过  fs.trash.checkpoint.interval  分钟后,会进入到回收站的检查点目录  `/user/<username>/.Trash/<date>`   ,在检查点目录的文件会在 fs.trash.interval 分钟后被彻底删除

使用hdfs dfs -rm -skipTrash可以跳过垃圾站,直接对回收站内的文件执行 rm 也可以永久删除文件




### 安全删除工具

大目录的[[hdfs 删除流程|删除操作]],会造成 namenode 持续无法响应请求的问题,所以对于大目录的删除,需要谨慎操作,可以考虑使用安全删除工具(hdfs-safe-delete-tool-1.0.jar):

1.删除前检测集群pending delete数量，如果超过某个阈值则等待其下去
2.满足1后执行递归的删除，删除一定批次的文件后，再次检测条件1
3.循环执行1-2直到文件清除完毕

