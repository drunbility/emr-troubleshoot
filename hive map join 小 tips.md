
#### map join hint

mapjoin 有两种实现: hint 语法 及 auto convert
在新版本中,默认是 auto convert,并且 hint 语法会被自动忽略,
由参数

`hive.ignore.mapjoin.hint `

控制,默认为 true ,也就是会忽略 hint 语法.

对于 chain map join ,即语法 “/*+ mapjoin(b,c)*/”  是支持的,但前提是上面的 ignore 要设置为 false.
对于默认行为 auto convert map join ,主要是由 auto convert 开关以及 smalltable 的大小阈值配置来控制的:
```
hive.auto.convert.join  控制是否开启
hive.mapjoin.smalltable.filesize  转换成 mapjoin 小表的阈值
```


上面两个配置是前置配置,适用于基础的两表 join 场景.
对于其他场景,比如多表join,即需要把两张表加载到内存当中,也就是 chain map join ,这个行为由

```
hive.auto.convert.join.noconditionaltask 配置来开启,默认是为 true.
该行为的 smalltable 的阈值由参数
hive.auto.convert.join.noconditionaltask.size控制
```

总的来说对于 auto convert join 的发生,主要是由小表大小是否满足阈值来触发的. 
此外还有一些其他控制逻辑,比如每张表满足配置的 size 阈值,会生成多个 map join,如果所有小表的大小之和满足 size 阈值,多个map join 会合并到一个 map join

#### map join hashtable

map join 首先要把小表读入内存:

![[Pasted image 20230227200112.png]]

中间首先由 `MapredLocalTask` 来读入小表生成 hash 表文件,这是一个本地任务,(如果是 cli 提交的,则任务启动在 cli 提交机器上,如果是连的 hs2 jdbc ,则任务是启动在 hs2 所在的集群上). 

####  MapredLocalTask

```
/**
 * MapredLocalTask represents any local work (i.e.: client side work) that hive needs to
 * execute. E.g.: This is used for generating Hashtables for Mapjoins on the client
 * before the Join is executed on the cluster.
 *
 * MapRedLocalTask does not actually execute the work in process, but rather generates
 * a command using ExecDriver. ExecDriver is what will finally drive processing the records.
 */
```


local task 有两种不同的提交方式,通过 `hive.exec.submit.local.task.via.child`  参数控制
通过单独的 java 进程提交或者直接运行在 client 进程中.默认为 true ,不另外启动  jvm 进程提交,这样省去了启动 jvm 进程的开销,但是容易发生内存不足的错误:

![[Pasted image 20230227201259.png]]


这个时候,可以调大  `hive.mapred.local.mem` 参数


#### [[ExecDriver 分析]]
