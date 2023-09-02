
### flume 调优的目标

1.  source, channel ,sink 三者能够达到一个平衡.也就是说 source 拉出来的数据, channel 能够放得下.同时 sink 能够及时,顺利把 channel 发来的数据写出去,而不致使得 channel 容量满了.
    
2.  上述三者的达到平衡时的数据吞吐和延迟能够满足业务的具体需求
    

假设该 flume conf agent 名称: agent1 ,source 名称:source1,channel 名称:c1, sink 名称:sink1

### kafka source 关键参数

-   agent1.sources.source1.batchSize 每批次往 channel 里写 kafka message 的最大条目数.这个值影响吞吐,值越大,吞吐量越大,channel 的压力也会相应的增大.
    
-   agent1.sources.source1.batchDurationMillis 每批次往 channel 里写 kafka message 的最大时间间隔,单位 ms.达到最大时间,无论上面的 batchSize 是否达到都会往 channel 里面写.这个值影响延迟,值越小,延迟越小,下游 channel 的处理速度也需相应的加快.
    

  

### memory channel 关键参数

-   agent1.channels.c1.capacity 这个值是 channel 能存储 message 的最大条目数,值越大,channel 能缓存的数据越多,这些数据从 source 中拉取,等待着被发往 sink 去.
    
-   agent1.channels.c1.transactionCapacity channel 一次事务能处理 message 的最大条目数, channel 的一个事务指的是从 source 里拉一次数据,或是往 sink 里发一次数据.值越大,每次处理的数据越多,但每次处理的时间会拉长,延迟升高.
    
-   agent1.channels.c1.byteCapacity 这个值也决定 channel 的容量,但不同于 capacity ,这个容量不是以 message 的条目数计,而是 channel 内所有 message 的 body 占的内存大小,以 baye 为单位计. 默认大小为 jvm Xmx 值的80%
    
-   agent1.channels.c1.byteCapacityBufferPercentage 一个百分比值,以 byteCapacity 大小的百分比例来指定所有 message 的 header 所占用的内存,默认20
    

  


### hdfs sink 关键参数

-   agent1.sinks.sink1.hdfs.batchSize 每批次往 hdfs 里写的 message 的条目数 ,值越大,吞吐越大,但处理速度会降低.
    
-   agent1.sinks.sink1.hdfs.rollInterval agent1.sinks.sink1.hdfs.rollSize agent1.sinks.sink1.hdfs.rollCount sink 写 hdfs 文件滚动规则设置,即按什么规则创建一个新文件.每隔多少时间,或者多少文件大小,或者消息条目数生成一个新的文件.值越小,文件滚动越快,也就是 sink 写文件的速度会越快,也会创建更多的文件
    

-   agent1.sinks.sink1.hdfs.threadsPoolSize sink 写 hdfs 的并发线程数,增大这个值,可以增加 hdfs sink 的性能,更高的吞吐和更低的延时
    
-   agent1.sinks.sink1.hdfs.maxOpenFiles sink 的并发增加时,如果同时处理的文件超过这个值,会关闭前面打开的文件.
    
-   agent1.sinks.sink1.hdfs.writeFormat = Text 如果文件需要被 hive 读取,需要这样设置.




---
[Flume中的HDFS Sink配置参数说明 – lxw的大数据田地 (lxw1234.com)](http://lxw1234.com/archives/2015/10/527.htm)
[Flume 1.9.0 User Guide — Apache Flume](https://flume.apache.org/releases/content/1.9.0/FlumeUserGuide.html#hdfs-sink)


