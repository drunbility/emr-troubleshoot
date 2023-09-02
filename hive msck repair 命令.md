
msck repair table  命令是为了修复 hive 分区的,一般来说,我们认为他是当 hdfs 上面新增了目录后,执行命令来添加分区的,但实际上这个命令后面还能指定选项

``MSCK [REPAIR] TABLE table_name [ADD/DROP/SYNC PARTITIONS];``

如果不指定后面的选项(==hive 3.x 才支持==),默认是 add partitions ,这样当删除 hdfs 上的分区目录时,不加 drop partitions 选项,就不能正确的修复 hive 分区元信息

如果使用  sync partitions 选项,相当于同时指定了 add 和 drop partitions 选项.

当一个表有大量的分区需要修复的时候,要注意 msck repair table 命令运行发生 OOM.可以先使用  msck table 命令先列出有多少元信息需要修复.

接着通过设置参数  `hive.msck.repair.batch.size`  通过分批次大小来执行修复,来避免发生 OOM,这个值默认是0,也就是一次执行所有的分区修复,所以容易导致 OOM




