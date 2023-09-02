
Offline Edits Viewer 是一种用于解析 Edits 日志文件的工具。当前的处理器主要用于不同格式之间的转换，比如原始的二进制格式和可读的 xml 格式

比如:
`hdfs oev -i edits -o edits.xml`

可读入 journalnode 目录下的 edit 文件,并转换成  xml 格式输出

可以在 xml 格式下对错误的事务记录进行修改,更正错误.根据回放报错日志中输出的 txid 可以定位到具体的 edit log 及错误的报错记录,可以定位具体有错误的操作记录,然后可以编辑 xml 文件,将 OPCODE 设置为 -1 ,将这条记录失效:

```

<RECORD>
    <OPCODE>-1</OPCODE>
    <DATA>
	    ......
    </DATA>
  </RECORD>
```


接着再将修复后的 xml 文件转换成二进制格式
`hdfs oev -p binary -i edits.xml -o edits`

替换到原 journalnode 目录,重新进行 edit logs 回放

---

oev 命令使用可以参考:
[Apache Hadoop 2.8.5 – Offline Edits Viewer Guide](https://hadoop.apache.org/docs/r2.8.5/hadoop-project-dist/hadoop-hdfs/HdfsEditsViewer.html)







