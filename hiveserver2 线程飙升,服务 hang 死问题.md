#### 现象

用户多次反馈他们的 hiveserver2 突然异常,业务侧连不上 hs2.

排查现场信息,发现日志中有如下报错:
![[Pasted image 20230304104754.png]]


#### 排查过程

初步怀疑是 hms 问题,但是 hive cli 能正常使用,说明目前 hms 正常,这个报错是短时性的错误.说明还是 hs2 的问题

查看监控,发现异常前 hs2 的处理线程数目持续飙升:

![[Pasted image 20230304104826.png]]


这个指标比较异常,于是开始分析 hs2 的堆栈信息.使用 arthas 工具打印火焰图下来分析,发现一个非常异常的宽塔:

![[Pasted image 20230304104844.png]]


hs2 中的大量线程卡在了这个语义分析步骤上了.不知道业务侧提交了什么复杂的或者递归的语句导致 hs2 编译这个语句 hang 住了.

查看  `HiveConf` 源码发现关于 hs2 编译 sql 语句有个参数可以配置编译超时时间.

`hive.server2.compile.lock.timeout`  这个参数默认值是0,就是永不超时.由于无法定位业务侧提交什么不合规的 sql 语句,所以把这个参数设置成300,5分钟超时间,来规避这个问题

`hive.driver.parallel.compilation`  设置为 true ,进行并发编译,可以防止某一个大sql编译耗时过长，阻塞其他sql执行









