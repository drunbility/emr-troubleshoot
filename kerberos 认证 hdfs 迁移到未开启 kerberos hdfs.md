#### 问题现象
[[hdfs 数据迁移|distcp]] 程序运行时报错

```
21/11/29 18:04:17 ERROR tools.DistCp: Invalid arguments:
java.io.IOException: Failed on local exception: java.io.IOException: Server asks us to fall back to SIMPLE auth, but this client is configured to only allow secure connections.; Host Details : local host is:  'node01-bigdata-prod-bj1.ybm100.top/10.33.75.5'; destination host is: "10.35.31.19":4007
```


#### 解决办法

distcp 命令中带上参数: -Dipc.client.fallback-to-simple-auth-allowed=true

使用  distcp 在这两个集群间迁移数据时,根据官方建议: [DistCp and Security Settings - Hortonworks Data Platform (cloudera.com)](https://docs.cloudera.com/HDPDocuments/HDP2/HDP-2.5.3/bk_administration/content/distcp_and_security_settings.html)

应该把 distcp 程序跑在 kerberos 认证集群上,反过来则不行, 会报错:

```
hadoop distcp -Dipc.client.fallback-to-simple-auth-allowed=true hdfs://HDP23:8020/test01.txt hdfs://HDP24:8020/ 
17/04/05 00:09:28 ERROR tools.DistCp: Invalid arguments: org.apache.hadoop.security.AccessControlException: SIMPLE authentication is not enabled. Available:[TOKEN, KERBEROS]
```




#### 补充

在非 kerberos 集群上访问 kerberos 集群时

1.要把 [[kerberos 认证]]配上,

2.要增加参数 -D hadoop.security.authentication=kerberos  但带上这个参数后就访问不了非 kerberos 集群了.

