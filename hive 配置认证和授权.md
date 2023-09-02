#### 场景说明
对于 hive 的认证和授权,首先要区分使用场景:

1.  hive 仅作为表格模式存储层.计算引擎直接调用 hive hcatalog api 来使用 hive,比如 impala,presto,spark-sql 等.这些组件的用户直接访问 hdfs 和 metastore server . hdfs 访问依赖本身提供的权限机制, metastore 访问权限在 hive 配置文件中配置
2.  hive 作为一个完整的 sql 执行引擎.这也是最普遍的使用场景.这种情况下也分为两种不同的小点
   - 用户使用 Hive Cli. 这些用户也是直接访问  hdfs 和 metastore 的.这种情况的权限机制跟第一大点的情况一样.这种情况已经过时了,新版本 hive 中不直接使用 hive  cli 了, 替换成 beeline cli .这就是第二种情况
   - 所有的用户统一通过访问  hiveserver2 来使用 hive,无论是使用 odbc ,jdbc ,beeline.访问用户都需要通过 hiveserer2 来访问数据,不能直接访问 hdfs 和 metastore . ^0d7cf0


#### 授权方案说明
总的来说,可配置三种不同的授权模式

1. 基于外部存储,配置 metastore server 认证授权.这种模式主要针对上面场景 1  和场景 2a .因为这两种场景用户可以直接访问 hdfs 和 metastore . hdfs 有自己的权限机制,所以这里只需额外配置  metastore server  的授权,metastore 的权限配置在 hive 配置文件中,配置比较复杂,较少使用.
2. SQL  标准的授权.  hiveserver2 提供了标准的基于 sql 语句的细粒度的权限控制,基于标准 sql 语句,配置非常灵活便捷,这种模式非常适合使用场景 2b
3. 使用第三方组件,比如 apache ranger ,他们提供了非常完备的大数据权限解决方案.覆盖所有场景

#### 认证方案说明

总的来说,hive 的用户认证方案也有四种:

1.  最安全的基于 kerberos 用户认证
2.  基于 LDAP 的 用户名/密码 认证方式
3.  自定义认证插件
4.  匿名认证,默认模式,允许认证任何人(也就是没有认证)


#### 方案选择

一般来说,选择最完备的认证方案:基于 kerberos 的用户认证加上基于 ranger 的授权. 能覆盖 hive 的所有使用场景,同时也能覆盖其他大数据组件的安全场景需求.对于 [[#^0d7cf0|2b]] 使用场景,用户没上 kerberos 和 ranger 组件,我们可以选择更加轻量级的方案:指定自定义插件来认证用户,使用标准 sql 授权语法来进行权限管理(详见附件)


---


[[HiveServer2配置认证和鉴权.pdf]]

[hiveCustomerAuthentication.zip](hiveCustomerAuthentication.zip)


