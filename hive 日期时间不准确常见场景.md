#### 原理说明

hive 的有两种常用的日期(时间)

date 类型  YYYY-­MM-­DD  例如  2022-01-10
timestamp 类型  YYYY-MM-DD HH:MM:SS  例如 2022-01-10 20:00:00
==用户使用日期时间类型的时候最容易忽略的一点就是日期时间类型隐性转换时带了时区信息,这是关键性的一点.==

当 date 或者 timestamp 被 select 出来,这些类型发生了隐式转换,转换成了 string 类型,并且不显示时区信息.这是 hive 日期(时间)不准确此类问题的本质原因.

#### 日期比较不符合日期

	![[Pasted image 20230217204524.png]]

这个 sql 比较结果看上去不对,但实际上是正确的,因为 b_25_start_c 是 timestamp 类型,他的时区信息是原始业务系统带过来的,是 UTC 时区,比 CST 时区慢了8个小时,可以通过函数来还原时区信息来验证:
date_format(b25__start__c,"yyyy.MM.dd G 'at' HH:mm:ss z")


#### 建表后日期不符合日期


![[Pasted image 20230217204554.png]]


这个结果看上去和预期查了一天,这是由于时区混乱了,hive server 的时区是 CST ,但是客户 cvm 的时区是 UTC.导致了函数计算的混乱.最好的方式是统一 hive server 和 cvm 的时区设置,如果不好操作,只能通过直接使用 string 类型来规避:create table dmt_production_cpq.tmp_min_date as select cast(date_add('2021-12-01',-1) as string) as min_date;

#### 物理节点时区发生变动

客户反馈 hiveSQL任务中，from_unixtime函数将时间戳转换为日期，前后不一致；同一个任务，在对时间戳进行转换后，可能会相差8小时 ，偶现
客户已经意识到了时区的设置统一,他们 hive server2 和 cvm 的时区都设置成了 UTC ,但是还是会偶尔出现结果相差8小时的现象.后来排查发现他们会动态扩缩容 task 节点,task 节点默认是 CST 时区,所以当 mr task 跑到了 task 节点上时,会出现相差8小时问题.后续客户在 task 节点要在引导脚本中加上时区修改逻辑.

