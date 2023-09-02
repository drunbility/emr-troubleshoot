

expect是一个自动化交互套件，主要应用于执行命令和程序时，系统以交互形式要求输入指定字符串，实现交互通信。

expect自动交互流程：

spawn启动指定进程---expect获取指定关键字---send向指定程序发送指定字符---执行完成退出.

注意该脚本能够执行的前提是安装了expect
`yum install -y expect

```
#!/usr/bin/bash

/usr/bin/expect << EOF

spawn hdfs namenode -recover

expect {

"Are you ready to proceed? (Y/N)" {send "y\n"}

}
expect eof

EOF

```  

---
#shell 

[Linux expect 介绍和用法一 - 梦徒 - 博客园 (cnblogs.com)](https://www.cnblogs.com/saneri/p/10819348.html)





