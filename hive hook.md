



#### 部署
1. 上传 jar 到 hs2 节点 /home/hadoop 目录
2. hs2 中执行
   add jar /home/hadoop/hivehook-1.0.jar;
   set hive.exec.driver.run.hooks=com.tx.yc.hivehook.CosKeyValidateDriverHook;
   set hive.exec.pre.hooks=com.tx.yc.hivehook.CosKeyValidateExecHook;
   set hive.exec.post.hooks=com.tx.yc.hivehook.CosKeyValidateExecHook;

