##### staging 目录

`hive.exec.stagingdir`   配置,默认是 `.hive-staging` , 也就是在表自身目录下的 .hive-staging 目录,用于支持 hdfs 加密以及涉及到写入查询的临时结果文件存放.

一般来说要修改默认值,不应该放到和表自身目录下
```
	<property>
         <name>hive.exec.stagingdir</name>
         <value>/tmp/hive/.hive-staging</value>
    </property>
```

这样有两个好处

- 便于[[hive staging 清理脚本|统一清理临时目录]],两种情况下 hive-staging 文件不会自动删除：1、任务执行过程中出现异常 2、长时间保持连接或者会话。
- 不作为表目录的子目录的话, move file 的效率更好,在大并发任务下更加明显 `org/apache/hadoop/hive/ql/metadata/Hive.java:3118`


#### scratch 目录

`hive.exec.scratchdir`   默认值是 /tmp/hive ,hdfs 上的临时目录根目录,对于每一个客户端用户,会创建 `${hive.exec.scratchdir}/<username>`  目录,存放非查询结果内的临时文件,比如 job plans file

`hive.exec.local.scratchdir`  默认值是 `${system:java.io.tmpdir}" + File.separator + "${system:user.name}`     [[java 命令行工具#^940c6b]]  本地的临时目录,一般用于存放 hive server2 的临时文件,比如 hs2 session 临时目录


#### 配置 scratchdir 清理服务

客户端异常结束情况下, scratchdir 目录不会清理,久而久之,会占用很大存储空间,可以配置清理服务 ^a01142

`hive.server2.clear.dangling.scratchdir`  设置为 true ,启用`cleardanglingscratchdir` 服务定时清理挂起的临时目录
`hive.server2.clear.dangling.scratchdir.interval`  定时清理的间隔,默认 1800s
`hive.scratchdir.lock`   需设置为 true ,锁定正常的临时目录,避免被清理服务清理掉正在使用中的临时目录

清理服务运行过程中,会先要判断临时目录是否在使用中,也就是是否存在 Lock 文件,如果没有锁文件,则会跳过该目录,所以对于历史遗留的 scratchdir ,清理服务无法进行清理,需要人工去判断这些目录是否还在使用,并进行清理

也可以通过命令手动执行一次清理服务
`hive --service cleardanglingscratchdir`





 

 


