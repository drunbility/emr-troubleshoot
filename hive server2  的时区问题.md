#### 关于时区

GMT 是旧的世界标准时,UTC 是新的世界标准时,他们之间的时间数值几乎没有差别,可以认为是一样的

machine time 机器时间，一般是用 System.currentTimeMillis() 方法获得的毫秒数就是机器时间,表示的是从 epoch 时间（1970年1月1日0时0分0秒）开始到现在已经过了多少毫秒. 从 epoch 时间 算起,过了这些毫秒时间 ` System.currentTimeMillis()` 后的时间(年月日时分秒),就是  UTC 时间.
UTC 时间就是世界标准时间,不受时区影响,在全球任意地方都表示同一时刻. 
UTC 时间是一种 human time,也就是人类可以方便查看的时间形式

平常所说的北京时间叫本地时间,本地时间是在UTC时间的基础上,通过加减所在的时区位置得到的
比如 CST 时间(China Standard Time，中国标准时间),是先获取 machine time，根据 machine time 算出 UTC 时间，然后再在UTC时间基础上加8个小时，算出中国境内的本地时间,即UTC+8,本地时间是另外一种 human time

linux 中本地时区设置:
```
ln -sf /usr/share/zoneinfo/UTC /etc/localtime
```

可以通过 timedatactl 命令查看上述时间:

![[Pasted image 20230207200240.png]]


#### hiveserver2 时区

如果要指定 hive server2 的时间,可以在 jvm 启动参数中增加:
`-Duser.timezone=Asia/Shanghai` 或者 `-Duser.timezone=UTC`

这样在 hiveserver2 中一些得到的默认时间就是 UTC 时间,和系统的本地时间有差异了

![[Pasted image 20230207201104.png]]


#### hive server2 timestamp 类型

在 hive 中 timestamp 本质上是 Unix 时间戳,是机器时间.
在使用 beeline 连接到 hs2 中,要特别注意,如果 hs2 启动参数中指定了时区信息,我们在执行 
`select current_timestamp();` 
返回的结果会发生隐式转换,变成了带上了时区信息的本地时间格式,**如果在 hs2 启动参数中指定了和系统不同的时区信息**,这样在做日期时间字符串比较的时候非常容易出现[[hive 日期时间不准确常见场景|不符合预期的结果]]:

![[Pasted image 20230207210712.png]]
看上去是现在的日期时间小于过去的日期时间

但其实这个时间戳是准确的 unix 时间戳

![[Pasted image 20230207210955.png]]
代表的时间是北京时间 `2023-02-07 21:08:19`  


可以说 timestamp 类型需要去关联一个时区信息后才能变成可读的字符串形式,也就是 human time.






  


 
----
[有关机器时间、UTC时间、本地时间的总结 - 腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1513057)

