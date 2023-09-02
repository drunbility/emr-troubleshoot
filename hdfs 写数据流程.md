#todo


#### 重要概念

-   1个block由多个chunk data组成, 每个chunk data里存储的都是二进制数据
-   1个block对应有一个 meta 文件, 存放与之对应的chunk checksum
-   HDFS组件间传输数据最小以packet 为单位, 1个packet由多个chunk构成, 传输后取出其中的chunk (data+checksum), 最后构成 block 和 meta ^9bf1af

![[Pasted image 20230209104300.png]]

![[Pasted image 20230209104305.png]]



**总结:** block是HDFS**操作**文件的最小单位, 它由chunk为单位**组成**, 但由packet为单位进行**传输** , 三个概念都是读写过程中经常遇到的, 十分重要.


#### 整体流程


![[Pasted image 20230212141131.png]]


#### client 侧流程
发包线程 `DataStreamer` 是 client 端代码核心,这个后台线程类的`run()`方法中, 线程启动是在 `create()` 调用最后, 也就是说还没开始`write()`的时候, 它就已经在后台运行了, 核心做**两件事**:
-   是唯一可以控制开/关通往DN的通道(pipeline)的线程
-   负责执行几乎所有正常写入, 以及失败/错误恢复的操作

![[Pasted image 20230212141308.png]]


#### namenode 侧流程
`FSNamesystem` 是 namenode 端的代码核心,主要完成了:
- 创建文件
- 创建 block
   1. 检查
   2. 选择 DN
   3. 更新元信息
- 完成文件

#### datanode 侧流程 Pipeline

datanode 侧构建了核心的数据流管道 pipeline,也即完成了数据的实际写入落盘过程

![[Pasted image 20230212153507.png]]


客户端在创建pipeline的时候, 会发送一个火车头(Sender)给第一个DN, 调用它的`writeBlock()`方法序列化一系列的DN信息, 然后再DN这边, 则通过启动之后就通过`Receiver`的具体实现`DataXceiver`的线程来接收信息了, 但是一个DN可能会有许多个Client同时发送**读写**请求, 这里DN就提供了一个控制层来处理(如图所示):

![[Pasted image 20230212161618.png]]


其中 DataXceiver 中的核心方法是  `writeBlock` , 方法里面核心的处理类是 BlockReceiver ,正是他处理了传输的 packet [[#^9bf1af]]

![[Pasted image 20230214204202.png]]

跟 DataStreamer 处理过程类似
