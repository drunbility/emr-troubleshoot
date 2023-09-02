
### 问题现象

在条件查询字句 where 字句中,如果使用了特殊的 udf 函数,会导致过滤条件不会生效.

- 案例一
```
#正常写法:
select 
case when reflect('java.net.URLDecoder', 'decode', flowpond) is not null
                    then  str_to_map(reflect('java.net.URLDecoder', 'decode', flowpond), '&', '=')
                else map() end ['pvid'] as imp_pvid,
           sub_trade_id
from dataware.dwd_trade_course_df 
where  dt = '20220820'  
limit 5;

#异常写法:
select 
case when reflect('java.net.URLDecoder', 'decode', flowpond) is not null
                    then  str_to_map(reflect('java.net.URLDecoder', 'decode', flowpond), '&', '=')
                else map() end ['pvid'] as imp_pvid,
           sub_trade_id
from dataware.dwd_trade_course_df 
where 
(case when reflect('java.net.URLDecoder', 'decode', flowpond) is not null
                      then  str_to_map(reflect('java.net.URLDecoder', 'decode', flowpond), '&', '=')
                  else map() end ) ['pvid'] is not null and  dt = '20220820'  
limit 5;


```
异常写法会导致分区过滤条件失效,造成全表扫描

- 案例二
```
#异常写法
select
		user_id,
		sub_trade_id,
		str_to_map(
			reflect('java.net.URLDecoder', 'decode', flowPond),
			'&',
			'='
		) [ 'fuid' ]  as fuid
	from
		dataware.view_dwd_trade_course_df
	where
		dt = '20230314'
		and str_to_map(
			reflect('java.net.URLDecoder', 'decode', flowPond),
			'&',
			'='
		) [ 'fuid' ] != ''
		and trade_status=1 limit 10000;

#正常写法
select
    user_id,
    sub_trade_id,
    flowid
from
    (select * ,
    str_to_map(
        reflect('java.net.URLDecoder', 'decode', flowPond),
        '&',
        '='
    ) [ 'fuid' ]  as flowid
    from 
    dataware.view_dwd_trade_course_df
    ) a
where
    a.dt = '20230314'
    and a.flowid != ''
    and a.trade_status=1 limit 10000;

```

### 排查过程

比较正常写法和异常写法的执行计划,可以看到,主要差异是 tablescan 阶段有没有生成过滤表达式

![[企业微信截图_14c860e8-5059-4f44-aebb-a5a8a5466036.png]]


![[企业微信截图_e7fd370c-fa9f-4a10-a67a-f6b39f66f375.png]]

现象是在 where 子句中使用某些特殊的  udf 函数,会导致过滤条件失效,从而造成全表扫描



### 问题结论

跟踪源码并进行 debug ,确认了执行计划优化过程中确实是因为filter带有udf导致filter下推失效。这个是很早的一个jira加的判断，社区最新的实现也还是保持这个逻辑.

`org.apache.hadoop.hive.ql.ppd.ExprWalkerInfo#isDeterministic`

![[企业微信截图_4db0c919-971d-490a-b1bd-8ce14fbb8260.png]]

非常边缘的 case ,这种应该让业务改写成正确的 SQL 写法来解决.

---
[HIVE-23893/HIVE-14661,split filter into deterministic filter which will be push down and nondeterministic filter which keeps current position by letsflyinthesky · Pull Request #1322 · apache/hive (github.com)](https://github.com/apache/hive/pull/1322)

[HIVE-23893: Extract deterministic conditions for pdd when the predica… by dengzhhu653 · Pull Request #1308 · apache/hive (github.com)](https://github.com/apache/hive/pull/1308)






