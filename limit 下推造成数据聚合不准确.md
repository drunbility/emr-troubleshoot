### 问题描述

```
select kv['name'],count(i) as pv from zuoyeapp_offline_action_hour where dt='20220206' and dt=indt and hour='20' group by kv['name'] limit 10;
```

用户反馈这个语句跑出来的结果不符合预期
![[wecom-temp-40545-3ca9c03b42fc0a815eb25e996d03416f.png]]
用户直接拿这个查询结果做条件查询技术得出来的 pv 值远远大于上面的值
并且只有使用mr引擎的时候发现有这个问题，spark引擎没有发现



### 排查过程
#todo 


### 结论

问题原因是因为limit operator被下推到map 端 partial  aggregation导致 reduce端的 final aggregation的数据不是全量数据，结果聚合的结果错误，hive社区已经修复这个问题，详见[[HIVE-24579] Incorrect Result For Groupby With Limit - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/HIVE-24579)，解决方法为backport这个patch




