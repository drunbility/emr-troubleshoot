#spark

### Task

Task是Spark中最小的执行单元。 spark 中的任务执行一系列指令。例如。读取数据、过滤和在数据上应用 map() 可以组合成一个任务。任务在执行器  executor  内部执行。


### Stage

一个阶段包含多个任务，阶段中的每个任务都执行相同的指令集。

### Job
一项工作包括几个阶段。当 Spark 遇到需要 shuffle 的函数时，它会创建一个新阶段。 reduceByKey()、Join() 等转换函数将触发洗牌并产生新阶段。当读取数据集时，Spark 还会创建一个阶段。

### Application

一个应用程序包含多个作业。每当您执行像 write() 这样的操作函数时，就会创建一个作业。

### Summary

A Spark application can have many jobs. A job can have many stages. A stage can have many tasks. A task executes a series of instructions.

![[Pasted image 20230106112219.png]]




---
[What are applications, jobs, stages and tasks in Spark? – Hadoop In Real World](https://www.hadoopinrealworld.com/what-are-applications-jobs-stages-and-tasks-in-spark/)
[What is the concept of application, job, stage and task in spark? - Stack Overflow](https://stackoverflow.com/questions/42263270/what-is-the-concept-of-application-job-stage-and-task-in-spark)


