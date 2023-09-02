
#### 停顿时间和吞吐量


在 jvm 中,gc 吞吐量指的运行用户代码的时间与 CPU 总的时间的比值,即吞吐量=运行用户代码时间 / 运行用户代码时间+垃圾收集时间,虚拟机总共运行了 100 分钟,其中垃圾收集花掉 1分钟,那么吞吐量就是 99%

垃圾收集时间即是  gc time ,垃圾收集暂停时间即是 gc pause time .
gc pause time 降低(也就是低延迟)必然是以 gc time 时间变长(吞吐量降低)为代价的.

所以,在 gc 场景中,低延迟和高吞吐是不可兼顾的.现在容易获取多核 cpu 的计算机硬件条件以及互联网的用户场景下,一般来说会牺牲吞吐来获得较低延迟.






