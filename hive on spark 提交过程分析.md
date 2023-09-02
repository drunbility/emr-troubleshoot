#todo 



### Driver run task 流程

起点在 Driver execute() 方法里 1839 行:
`org.apache.hadoop.hive.ql.Driver#launchTask`

launchTask 会根据配置 `hive.exec.parallel` 决定是否并行执行 task
接着会走到方法 `org.apache.hadoop.hive.ql.exec.TaskRunner#runSequential`
里面调用抽象类 Task 方法 `org.apache.hadoop.hive.ql.exec.Task#executeTask`
这里会真正开始执行子类 Task 的 方法

![[Pasted image 20230330202924.png]]

execute 方法调用到不同的子类 Task 实现里面,对于 SparkTask ,则是执行到方法

`org.apache.hadoop.hive.ql.exec.spark.SparkTask#execute` 内

这里正式开始 SparkTask 的执行逻辑




### SparkTask 提交流程


接着看 `org.apache.hadoop.hive.ql.exec.spark.SparkTask#execute` 方法






[[spark yarn-client 和 yarn-cluster]]



### 常见问题


yarn am 与  hivesever2 中的 spark client 间连接超时
