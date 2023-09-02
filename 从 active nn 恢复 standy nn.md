standby nn由于各种原因,比如[[edit logs 错误导致 hdfs 无法重启|edit logs 回放失败]],启动失败，active nn正常，从active nn恢复standby nn

1. 在控制台暂停standby nn，防止后台自动拉起
2. 进入active nn所在的机器，进入hadoop用户，[[长期未完成 checkpoint 导致重启 namenode 慢问题#^d799b1|做一下checkpoint操作]]
3. 进入standby nn所在的机器，进入hadoop用户，备份元数据目录，新建current，并将VERSION复制过去（如果VERSION文件不存在，可以去nn节点copy过来）
```shell
cd /data/emr/hdfs/namenode/
mv current current_bak
mkdir current
cp current_bak/VERSION current
```

4. 初始化standby nn节点
5. 控制台启动standby nn

