
### 执行报错常见问题
使用 jmap jstack jinfo 等工具排查 Java 进程的时候,有两个注意点:
1. 需要先切换到进程的所属用户去执行命令
2. 要注意 java 临时目录是否被清理了,如果被清理了,一般会报错 
   `Unable to open socket file: target process not responding or HotSpot VM not loaded`


补充说明:
OpenJDK 的命令行工具有一个实现 [findSocketFile](https://cr.openjdk.java.net/~sla/7132199/webrev.00/src/solaris/classes/sun/tools/attach/LinuxVirtualMachine.java.lhs.html#anc2) 去寻找套接字文件,套接字一般在 `java.io.tmpdir` (jvm 通过这个参数获取操作系统缓存的临时目录)下，Linux 在`/tmp/.java_pid<pid>` 这个文件 是被 `systemd-tmpfiles` 这个程序定时清理掉的。 ^940c6b

如需要彻底解决这个问题,可以 `systemd-tmpfiles`  服务配置
以 Centos 7 为例
修改/usr/lib/tmpfiles.d/tmp.conf，添加
`X /tmp/.java_pid*`


### 使用注意事项

- `jmap -histo:live  pid` 命令会触发 Full GC ,在 gc log 中会输出相关信息

```
 [Times: user=1.01 sys=0.06, real=0.10 secs] 
2022-12-20T16:33:59.303+0800: 968571.810: [Full GC (Heap Inspection Initiated GC)  25291M->4313M(24576M), 8.4499764 secs]
   [Eden: 15896.0M(17664.0M)->0.0B(14744.0M) Survivors: 104.0M->0.0B Heap: 25291.8M(31520.0M)->4313.6M(24576.0M)], [Metaspace: 414153K->413397K(446464K)]   
```









---


[Troubleshooting · jvm-profiling-tools/async-profiler Wiki (github.com)](https://github.com/jvm-profiling-tools/async-profiler/wiki/Troubleshooting)









