

-   查找并删除临时文件
  `ll -rt /tmp | wc -l
  `find /tmp -type f -exec rm {} \;

-   删除0字节的文件
  `find /home -type f -size 0 -exec rm {} \;

-   查找包含大量文件的目录并删除
```
for i in /*; do echo $i; find $i | wc -l ; done
for i in /var/*; do echo $i; find $i | wc -l; done
```

-   参数列表过长,大目录分批次删除
  `ls | xargs -n 500 rm -rf #-n表示一次传递参数的个数`


- 查询处于 deleted 状态的文件,此时还有运行中的进程持有被删除文件的句柄,这些文件占用的磁盘空间没有被释放 
  `lsof | grep deleted` 
  `lsof | grep -c DEL`
  找到持有这些文件的 pid ,然后 kill pid 释放文件 ^6396f2

- ll /proc/{nodemanager的进程id}/fd   |grep delete 
  通过这个方式找到删除的句柄，通过 >/proc/{nodemanager的进程id}/fd/2099置空对应的文件也可以不重启情况下释放,2099是对应的文件id
  
