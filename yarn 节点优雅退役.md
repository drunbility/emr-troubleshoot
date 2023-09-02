
yarn 节点下线的时候可以指定优雅下线模式,也就是指定等待时间,等待节点上的正在运行的任务完成后,再下线这个节点

也就是说在执行下线命令的时候指定 `-g` 参数
`yarn rmadmin -refreshNodes [-g [timeout in seconds] -client|server]`

`-g`  指定超时时间

`-client|server`  指示超时跟踪是否应由客户端或 ResourceManager 处理。客户端跟踪是阻塞的，而服务器端跟踪不是。

YARN 节点的优雅停用是一种停用 NM 同时最大程度地减少对正在运行的应用程序的影响的机制。一旦节点处于 DECOMMISSIONING 状态，RM 将不会在其上安排新容器，并将等待运行的容器和应用程序完成（或直到超过退役超时），然后再将节点转换为 DECOMMISSIONED。

---
[Apache Hadoop 3.3.5 – Graceful Decommission of YARN Nodes](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/GracefulDecommission.html#configuration)


