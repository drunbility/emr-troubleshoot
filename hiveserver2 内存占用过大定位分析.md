#### 问题和背景

用户反馈 hiveserver2 连接不上,影响使用.登陆控制台,集群事件里显示 hiveserver2 一直在告警 full gc. 建议客户调整 hiveserver2 的 jvm 参数.客户不认可,客户认为他就一个查询,为什么会造成 hiveserver2 full gc.认为 hiveserver2 连不上不是内存占用的问题,需要我们给出令他信服的问题原因和解决措施.


#### 定位原因

使用 arthas 工具,来实时查看  hiveserver2 的进程状态:

-  登陆 hiveserver2 机器, 切换到 hadoop 用户,上传 arthas 全量包 arthas-packaging-3.5.4-bin.zip 到 /home/hadoop 目录,unzip 解压
-  ps -ef |grep HiveServer2  找到 hiveserver2 进程 id
-  切换到 hadoop 用户,执行命令 java -jar  arthas-boot.jar  ,会提示选择序号,键入 hiveserver2 进程 id 对应的序号,之后 arthas 工具就绑定到了这个进程
-  接着执行命令 dashboard ,查看进程实时数据面板,可以看 usage 那一列(当时那一列都是90%以上,并且第一栏线程列表中,NAME 那一列有很多 GC 的进程)
-   通过进程实时面板基本就确定了 hiveserver2 连不上的原因就是长时间的 full gc 导致的
-   注意,一定要使用 stop 退出 arthas 挂载,没退出的话时间久了会把这个进程打挂


####  hiveserver2 full gc 的根因

一般 hiveserver2  频繁 full gc 的原因是客户端连接太多了,但客户不认可这个原因,因为他认为就他一个连接.为了定位 hiveserver2 full gc 的原因,所以需要把 java heap dump 下来分析

-  同样使用 arthas 工具,跟前面操作一样,先绑定到当时出问题的 hiveserver2 进程
-  执行命令 heapdump /tmp/dump.hprof  ,等待 dump 完成,把 dump.hprof 文件下载下来
-  分析 hprof 文件. hprof 文件可以直接使用 idea 工具打开,按两下 shift 弹出搜索框,输入 hprof ,然后选择 open ,打开下载下来的 hprof 文件
![[Pasted image 20230207150951.png]]

- 点开 Biggest Objects ,点击 Calculate ,查看对象视图,可以看到一个异常的 arraylist,占用非常大:
![[Pasted image 20230207151010.png]]

- 继续点开这个 list 的树,定位该 arraylist 源码:
![[Pasted image 20230207151026.png]]

- 查看源码,搜索该 list 的引用 :
![[Pasted image 20230207151129.png]]
-   定位到是这个  list 里面存放了上百万个元素,并且没有释放.
-   继续展开这个 list,最终的占用是
![[Pasted image 20230207151343.png]]

-   这个 list 里面保存了很多个 s3a 的地址,所有的地址加起来大小达到 1.8 G. 
-   猜测有两种可能:  
    1.  客户使用 dbeaver 工具连接 hive,可能没有正确关闭连接,发生了内存泄漏,导致该 list 内的元素不停的堆积.  
    2.  客户查询的这个表有非常多的小文件,导致 hive 做 split 的时候保存了所有小文件的 split 的. 这种可能性比较大.

