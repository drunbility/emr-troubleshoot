
修改默认目录后:
```

# !/bin/bash
# delete hive-staging filess except today
hive_dir="/tmp/staging/"
file_list=`hdfs dfs -ls ${hive_dir}|grep -v $(date +%Y-%m-%d) | grep -v $(date -d last-day +%Y-%m-%d) | awk '{print $8}'`
hdfs dfs -rm  -r  ${file_list}
```



清理默认目录:

```
# !/bin/bash
# 找出已存在的hive-staging文件且删除脚本 
# 找出的临时文件列表存放到/tmp/hive_file.txt
 
dir_list="/usr/hive/warehouse"
> /tmp/hive_file.txt
touch /tmp/hive_file.txt
for file_path in `hadoop dfs -ls ${dir_list} | awk '{print $8}' `
do
hdfs dfs -ls $file_path |grep "hive-staging_hive" | awk '{print $6,$8}' &gt;&gt;/tmp/hive_file.txt
for file2_path in `hdfs dfs -ls $file_path| awk '{print $8}'`
do
hdfs dfs -ls $file2_path |grep "hive-staging_hive" | awk '{print $6,$8}' &gt;&gt;/tmp/hive_file.txt
done
done
 
hive_stag_list=`cat /tmp/hive_file.txt|grep -v "items"|grep -v $(date +%Y-%m-%d)|grep -v $(date -d last-day +%Y-%m-%d)|awk '{print $2}'`
hdfs dfs -rm -r ${hive_stag_list}


```

