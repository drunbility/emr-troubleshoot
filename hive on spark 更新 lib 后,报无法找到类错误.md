
#### 现象

在更新 hive lib 后,用户反馈查询报错
```
INFO: 2022-07-13 11:12:34:0734 Taskname: f_9e543980baf011ecb9f773b6b0dcdc5c RunIndex: 20220712 failed due to queryengine error: Query Failed: org.apache.hive.service.cli.HiveSQLException: Error while processing statement: FAILED: Execution Error, return code 3 from org.apache.hadoop.hive.ql.exec.spark.SparkTask. org.apache.hive.com.esotericsoftware.kryo.KryoException: Encountered unregistered class ID: 460
Serialization trace:
realInput (org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat)
parentOperators (org.apache.hadoop.hive.ql.exec.JoinOperator)
reducer (org.apache.hadoop.hive.ql.plan.ReduceWork)
right (org.apache.commons.lang3.tuple.ImmutablePair)
edgeProperties (org.apache.hadoop.hive.ql.plan.SparkWork)
at org.apache.hive.com.esotericsoftware.kryo.util.DefaultClassResolver.readClass(DefaultClassResolver.java:137)
```
看上去是更新的 lib 有问题

#### 结论

排查[[hive on spark 提交过程分析|源码]]后,发现:

hiveServer2在submit spark application后,会调用这个函数让spark去更新资源文件,这个函数里会让spark driver去add jar
因此,hiveserver2没有重启,那么这里他去add jar的时候就会缺少新增加的包,但是spark启动的时候,已经是用了新的jar包启动的,导致缺少jar包


![[企业微信截图_facefcc6-a73d-48de-bc9a-873a712b14c7.png]]


![[企业微信截图_610f5caa-8907-46b6-b27f-805c7d5e149d.png]]



