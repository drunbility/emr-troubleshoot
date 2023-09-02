
#### 现象
hms 日志会打印错误日志
```
Caused by: com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure

The last packet successfully received from the server was 1,138,152 milliseconds ago.  The last packet sent successfully to the server was 1,138,154 milliseconds ago.

```


#### 解决办法

这种情况是 hms 中元数据库连接池中线程数目不够导致的,可以调大这个参数
`datanucleus.connectionPool.maxPoolSize`
这个参数默认值是10 ,可以适当调大



