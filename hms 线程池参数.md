
hms 使用 datanucleus orm 框架(基于 JPA 实现), datanucleus 有一些比较实用的配置

- datanucleus.connectionPool.maxPoolSize  默认值是10,对于大规模繁忙集群来说过小了,可以调大到 50 
- datanucleus.connectionPoolingType  框架使用的数据库连接池类型, hive2 中默认是 `BONECP` ,hive3 中默认为 `HikariCP`
- datanucleus.transactionIsolation  设置连接数据库的[事务隔离级别](https://blog.csdn.net/qq_33290787/article/details/51924963?spm=a2c6h.12873639.article-detail.3.349c61f0sDuNSR),默认是 read-committed




---
[Hive Metastore Server生产化实践 | Haihua's blog (ericsahit.github.io)](https://ericsahit.github.io/2016/11/18/Hive-Metastore-Server%E7%94%9F%E4%BA%A7%E5%8C%96%E5%AE%9E%E8%B7%B5/)


