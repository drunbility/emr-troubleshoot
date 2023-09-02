
排查[[hs2 本地内存泄漏问题|本地内存泄漏]]的一种方式

1. 配置 dump 文件路径和 dump 规则
![[Pasted image 20230222220024.png]]
每分配 1g 内存 ,dump 一个快照到指定路径下

2. 配置 tcmalloc 启动路径
![[Pasted image 20230222220122.png]]


3. 分析 dump 路径下的 dump 文件
`pprof --pdf  java hiveserver2.xxxx.hprof > xxx.pdf`


4. 上面成功配置后,Java 程序启动起来后,这种 so 的包不能随便修改,否则会导致 java 进程直接宕机,并且不会留下任何异常日志信息

---
[一次大量 JVM Native 内存泄露的排查分析（64M 问题） | HeapDump性能社区](https://heapdump.cn/article/3530243)


