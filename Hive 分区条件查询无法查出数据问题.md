
### 现象
用户反馈语句
`select * from  ugc.app_activation_attribute_cpa  where  app = 'pinyin' and dt='20220301' and hour = '00' limit 1 `
这个语句能在 presto 中查询出结果,但是在 hive 中不能查出结果


### 分析过程

首先分析表结构和分区目录,发现这个分区 app = 'pinyin' and dt='20220303' and hour = '00'  和分区  app = 'pyAPP' and dt='20220303' and hour = '00'  共享同一个 cos 目录

`/hive/ugc/app_activation_attribute_cpa/app=pyAPP/dt=20220303/hour=00`

并且排查出现象 :
```
select * from  ugc.app_activation_attribute_cpa  where  app = 'pinyin' and dt='20220301' and hour = '00' limit 1  不能查出数据,presto 能查出
select * from  ugc.app_activation_attribute_cpa  where  app = 'pyAPP' and dt='20220301' and hour = '00' limit 1  能查出数据

```

关键是 `pinyin` 和 `pyAPP` 两个比较特别的分区,和用户侧了解到 pyAPP是 flink 生成的，pinyin 是通过alter table add partition 的方式添加的,且指向了同一个存储目录

首先用户侧肯定是要规避这种用法的.不同的分区应指向不同的存储目录.这个问题的肯定是由于这种操作引发的.


### 结论

最终排查发现还是这个分区下面的数据的特殊性质造成的.这个 `pinyin` 分区下面的数据是 parquet 格式,这些数据是带 schema 的,并且数据的 schema 实际是 `app=pyAPP` ,


hive 在执行 `select * from  ugc.app_activation_attribute_cpa  where  app = 'pinyin' and dt='20220301' and hour = '00' limit 1` 查询的时候,做了谓词下推,也就是将 `app='pinyin'` 判断条件下推到了 parquet 文件,那么这些数据文件自然就被过滤掉了.

这个从 hive cli debug 信息中也能证实:
![[wecom-temp-159172-da48ee3c48df3aee9f6e5b199df277a4.png]]


而 presto 中能查出,是由于 presto 对 `app = 'pinyin'` 条件只做了分区裁剪,并没有下推到数据.
在 presto 中 desc 这个表,发现 presto 把表 app 字段识别为分区字段了,所以并不会把这个谓词下推到 tablescan  operator 

![[Pasted image 20230330161430.png]]

实际查出来的数据应该是不准确的
![[Pasted image 20230330161049.png]]


而在 hive 中 desc 这个表, app 字段同时在普通字段内

![[Pasted image 20230330161928.png]]

所以 hive 在查询计划中做了谓词下推到 tablescan 算子,造成了这些差异


---
查看 parquet 文件工具:
`hadoop jar /home/hadoop/parquet/parquet-cli-1.10.1-runtime.jar org.apache.parquet.cli.Main meta /data/emr/hive/logs/part-1645431117744-4-16516`



