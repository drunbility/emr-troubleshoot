
mapreduce 任务:
```
task 参数
set mapreduce.map.memory.mb=3800;
set mapreduce.map.java.opts=-Xms3800m -Xmx3800m; 
set mapreduce.reduce.memory.mb=3800;
set mapreduce.reduce.java.opts=-Xms3800m -Xmx3800m;


增大MRAppMaster java heap
set yarn.app.mapreduce.am.resource.mb=30240;
set yarn.app.mapreduce.am.command-opts=-Xmx30000m;
```

hadoop 2 版本后参数的前缀都是 `mapreduce`, mapred 的前缀 `mapred.map.xxx`  是 hadoop 1 版本


mapreduce.map.memory.mb  是 yarn 给 container 的限制,如果不足会抛出错误:
	Container[pid=container_1406552545451_0009_01_000002,containerID=container_234132_0001_01_000001] is running beyond physical memory limits. Current usage: 569.1 MB of 512 MB physical memory used; 970.1 MB of 1.0 GB virtual memory used. Killing container.


mr 中 container 一般是一个 java 程序,并且是一个脚本通过 java 命令启动的,`mapreduce.map.java.opts`   设置的便是 java 启动命令中的 -Xmx  heap size 参数,如果不足会抛出错误:
	Error: java.lang.RuntimeException: java.lang.OutOfMemoryError


 一般来说,  内层的 heap size 值不大于外层的 container 大小限制





