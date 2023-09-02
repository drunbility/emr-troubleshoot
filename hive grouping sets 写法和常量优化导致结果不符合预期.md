
#### 现象

问题SQL:

```
with tbl_mid as (  
select  
'test_value' as test_field,  
*  
from  
default.codis_ykbzrdd  
where dt= '20220815'  
and `type` in ('string', 'list', 'set', 'hash') limit 100  
)  
select  
`type`,  
nvl(test_field, 'default_test_value'),  
count(*)  
from  
tbl_mid  
group by  
`type`,  
test_field grouping sets(type,test_field,(type,test_field)) limit 50;
```

执行该 sql 会生成不符合预期的结果:
![[Pasted image 20230309202835.png]]

必须把临时表拆开写:
```
create temporary table tbl_mid as  
select  
'test_value' as test_field,  
*  
from  
default.codis_ykbzrdd  
where dt= '20220815'  
and `type` in ('string', 'list', 'set', 'hash') limit 50;

select  
`type`,  
nvl(test_field, 'default_test_value'),  
count(*)  
from  
tbl_mid  
group by  
`type`,  
test_field grouping sets(type,test_field,(type,test_field));
```

才能生成符合预期的结果,关键差别在第二列:
![[Pasted image 20230309203249.png]]


#### 排查过程

首先比较执行计划,可以发现差异:
![[Pasted image 20230309203316.png]]


一个 key 被替换成了常量,怀疑是优化器做了优化导致的.接着在 HiveConf 类里面搜索 optimizer,果然找到一个优化器设置:

`HIVEOPTCONSTANTPROPAGATION("hive.optimize.constant.propagation", true, "Whether to enable constant propagation optimizer")`

将这个配置设置为 false 后,再继续执行 with as 写法的 sql ,就能生成预期的结果了.  
  
这个时候再去比较下关闭优化前后的执行计划,能看出关键差异:

![[Pasted image 20230309203652.png]]

在常量优化器关掉之前,NVL 函数并没有调用,直接被替换成了常量,关掉后,NVL 函数没有被替换.

又由于 `grouping sets` 的写法实际上是两个 `group by NULL,_col1` 和  `group by _col0,NULL` 的 union ,所以这个 NVL 的存在导致了最终结果的不同

 
#todo 这里像是一个 [[apache calcite]] 的 bug   [[CALCITE-1069] In Aggregate, deprecate indicators, and allow GROUPING to be used as an aggregate function - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/CALCITE-1069)






















