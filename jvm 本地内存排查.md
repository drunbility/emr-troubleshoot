
#### pmap 地址空间
接着 使用 `pmap ` 命令来获得内存地址空间的分配信息
![[Pasted image 20230222201202.png]]
最大 size 那一行就是系统分配给 jvm 的堆内存地址


#### 获得 core 文件
接着我们想要分析上面内存地址空间内具体的内存对象,可以使用命令 `gdp -pid pid`    工具来跟踪
但是这样会影响线上的程序,所以使用 gcore 工具产生core dump文件来独立分析. 
`gcore pid`
生成 core 文件过程中,会中断程序一段时间.接着会在执行目录下面生成 core dump 文件
接着可以使用  `gdb`  来查看 core 文件
`gdb TencentKona-8.0.3-unsafe_jfr/bin/java core.986310`
注意指定 java 的路径


#### 分析地址空间
比如要分析上图中 64376 这个大小的内存是什么.首先得到这段内存起始地址, 图中第一列 `7f04c0000000`  这个地址加上这段内存的大小 64376 ,就得到这段内存的结束地址
注意要使用十六进制来做加法(可以使用程序员模式的计算器):
- 64376 x 1024 = 65921024  
- 上面转换为 16进制 , 0x3EDE000 
- 起始地址加上 0x7f04c0000000  + 0x3EDE000 = 0x7F04C3EDE000 得到结束地址

根据上面的起始地址和结束地址,在上面的 gdb 工具的交互命令行工具内使用命令
`dump memory test.mem.bin 0x7f04c0000000  0x7F04C3EDE000`
![[Pasted image 20230222202223.png]]
会在当前目录下生成 test.mem.bin 文件,接着另起一个窗口,在这个目录下执行命令
`strings test.mem.bin  > test.mem.bin.log`
生成可读的 log 文件,接着就可以打开这个 log 文件来看这段内存里面是什么了



#### [[使用 tcmalloc 跟踪新增内存]]


#### tips
- 在排查 java 程序的本地内存问题的时候,应该做如下配置: -Xmx -Xms 配置成同样值 ,并且增加 -XX:+AlwaysPreTouch. 这样堆内存就固定不变了,变化的只能是堆外内存了.这样就能排除堆内的干扰




---
[Spring Boot引起的“堆外内存泄漏”排查及经验总结 - 美团技术团队 (meituan.com)](https://tech.meituan.com/2019/01/03/spring-boot-native-memory-leak.html)




