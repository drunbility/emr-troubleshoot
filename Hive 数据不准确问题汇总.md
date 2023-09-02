###  hive-mapjoin后数据不准确
版本：hive-2.3.x
引擎：hive on tez
现象：数据不准确
影响参数：hive.mapjoin.hybridgrace.hashtable=false
出现场景：

### hive-mapjoin后数据不准确
版本：hive-3.1.2
引擎：hive on tez
现象：数据不准确
影响参数： hive.convert.join.bucket.mapjoin.tez=false
出现场景：
社区支持：https://issues.apache.org/jira/browse/HIVE-20187

### hive-mapjoin后数据不准确
版本：hive-3.1.2
引擎：hive on tez
现象：数据不准确
影响参数： hive.optimize.joinreducededuplication=false
出现场景：

### hive-vector开启后数据不准确
版本：hive-3.1.2，hive-2.3.x
引擎：hive on tez
现象：数据不准确
影响参数：hive.vectorized.execution.enabled=false
出现场景：

### hive3 bucket分桶算法导致数据不准
版本：hive-3.1.2
引擎：hive on tez/mr
现象：数据不准确
影响参数：ALTER TABLE table_name SET TBLPROPERTIES ('bucketing_version'='1')
出现场景一：union all与left join混用
出现场景二：union all与insert overwrite混用
出现场景三：a left join b on a.id = b.id 在a,b都有满足条件的数据时，b表输出字段为null
问题说明：hash算法-murmur升级导致数据不准，目前解决方法降级

 

### hive字段类型不一致进行join
版本：hive-3.1.2，hive-2.3.x
引擎：hive on tez/mr
现象：数据不准确
影响参数：
出现场景一：select a.c1 ,b.c2 from a left join b on a.c1=b.c1，a.c1是字符串，b.c1是bigint
问题说明：不同的字段类型在hive进行语法解析时数据会被错误的转换

### hive-不同bucket.version的表进行关联
版本：hive-3.1.2，hive-2.3.x
引擎：hive on tez/mr
现象：数据不准确
影响参数：
出现场景一：a表的bucket.version是1，b的bucket.version是2，a left join b
问题说明：不同的字段类型在hive进行语法解析时数据会被错误的转换

### insert into 覆盖了历史数据
版本：hive-3.1.2，hive-2.3.x
引擎：hive on tez、mr
现象：数据不准确
影响参数：
出现场景：
insert into  \`tbl_name\`    as select xxx
问题说明：反引号 被解析成了insert overwrite

### cbo导致数据不准
版本：hive-2.3.x
引擎：hive on tez、mr
现象：数据不准确
影响参数：hive.cbo.enable=false
出现场景：not exists 无数据
问题说明：

### union all 后数据不准
版本：hive3.1.1
引擎：hive on mr
影响参数：
set hive.optimize.union.remove=true;
set mapreduce.input.fileinputformat.input.dir.recursive=true;
出现场景：union all,left jon

### union all 后数据无输出
版本：hive2.3.7
引擎：hive on mr/spark
影响参数：
set hive.optimize.skewjoin=false;
出现场景：union all,left jon
现象：数据无输出
客户1：高途 hive on mr 数据无输出
客户2：燃数 hive on spark 数据无输出

### subquery left jon 无法关联
版本：hive2.3.7
引擎：hive on tez
影响参数：
set hive.execution.engine=mr;
或
set hive.auto.convert.sortmerge.join=false;
出现场景：两个subquery left jon  
涉及patch：AMBARI-22464，HIVE-24907  
现象：右表数据关联不了


### [[limit 下推造成数据聚合不准确]]




### 通用排查思路
1. 按上面的列表快速试错  
2. 关闭优化参数（hive -e "set -v"|grep optimize|grep true） 列出所有优化有关的参数,然后逐个关闭