#ranger

ranger 对 hive udf 进行授权是 hive 3.0 才开始支持的,如果需要在 hive 2.3.7 中支持对 hive udf 的ranger 授权,可以合入社区 patch
[[HIVE-18841] Support authorization of UDF usage in hive - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/HIVE-18841)


在 ranger 中对 hive udf 进行授权时,要注意如果是创建临时函数,因为临时函数不属于任何 db,所以需要授予 global db 的权限,否则创建函数时会报没有权限的错误

![[企业微信截图_b362b559-01fb-49bc-a469-9996b4b78ecd.png]]

当创建永久函数的时候,就可以把 udf 权限 指定到具体的 db 了

