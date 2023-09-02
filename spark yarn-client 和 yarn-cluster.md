
spark on yarn 两种不同部署模式
这两种策略都是使用集群来分发任务；不同之处在于 driver  运行的位置：locally with spark-submit, or in the yarn cluster.

--deploy-mode  cluster/client


	Driver program :The process running the main() function of the application and
	creating the SparkContext






---
[Spark yarn cluster vs client - how to choose which one to use? - Stack Overflow](https://stackoverflow.com/questions/41124428/spark-yarn-cluster-vs-client-how-to-choose-which-one-to-use)



