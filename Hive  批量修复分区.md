#hive 

hive 迁移后,有时需要对所有表做分区修复,可以使用脚本来做[[hive msck repair 命令|批量修复]]

```
#!/bin/bash

set -e
mkdir -p debug

if [ ! -n "$1" ] ;then
echo "you have not input a file!"
else
echo "the word you input is $1"
fi

today=$(date "+%Y%m%d")
dataline=$(cat $1) ## 表列表
dbname=$1 ## 库名
runningnum=5 # 并发 session
num=0

for line in $dataline

do
num=$[$num+1]

echo $num

runningapp=$(ps -ef | grep BeeLine | grep -v grep |wc -l)

echo "runningapp:"$runningapp

while [ $runningapp -ge $runningnum ]

do

runningapp=$(ps -ef | grep BeeLine |wc -l)

echo "正在执行的任务数："$runningapp",sleep 3秒...."

sleep 3

done

echo ${dbname}.${line}

nohup beeline -u jdbc:hive2://localhost:7001/${dbname} -e "set hive.msck.path.validation=skip;set hive.msck.repair.batch.size=50;msck repair table ${dbname}.${line};" >> debug/${dbname}_${today}.log 2>&1 &

echo $line >> debug/${dbname}_${today}.list

#sed -i "1d" $1

sleep 3

#cd "$(dirname "$0")"

#str=`date +%Y%m%d%H%M%S`

done

wait

```


