读入拷贝目录,分目录做 distcp :

```
#!/bin/bash


if [ ! -n "$1" ] ;then
    echo "you have not input a file!"
else
    echo "the word you input is $1"
fi

dataline=$(cat $1)
var=$1
var=${var//,/ } #这里是将var中的,替换为空格
source=hdfs://nn03.pkx01.rack.xxxxx.com:xxxx
target=hdfs://HDFS84008
runningnum=20
export HADOOP_USER_NAME=ca_xxxx
export HADOOP_USER_PASSWORD=tx_xxxxx

for line in $dataline
do

    linedata=${line//,/ }
    
    path=""
    size=""
    mapnum=""
    taskname="ca_alg"
    eval $(echo $linedata | awk '{ printf("path=%s;size=%s",$1,$2)}')
    echo $path","$size","$taskname

    runningapp=$(yarn application -list|wc -l)
    echo "runningapp:"$runningapp

    while [ $runningapp -ge $runningnum ]
    do
      runningapp=$(yarn application -list|wc -l)
      echo "yarn正在执行的任务数："$runningapp",sleep 1秒...."
      sleep 1
    done

   
#小于10G map数 50
    if [[ $size -lt 10240000000 ]];then
      mapnum=50
      nohup    hadoop distcp  -Dmapreduce.job.name=${taskname} -Dmapreduce.reduce.memory.mb=4096 -Dfs.defaultFS=hdfs://HDFS84008  -Dyarn.app.mapreduce.am.resource.mb=8192 -prbugpt -i -strategy dynamic -skipcrccheck -numListstatusThreads 30 -skipcrccheck -update -delete -m $mapnum -bandwidth 100 $source$path $target$path > /dev/null 2>&1 & 

#大于10G小于100G，map数200
    elif [ $size -ge 10240000000 -a $size -lt 102400000000 ];then
      mapnum=200
      
     nohup    hadoop distcp  -Dmapreduce.job.name=${taskname} -Dmapreduce.reduce.memory.mb=4096 -Dfs.defaultFS=hdfs://HDFS84008  -Dyarn.app.mapreduce.am.resource.mb=8192 -prbugpt -i -strategy dynamic -skipcrccheck -numListstatusThreads 30 -skipcrccheck -update -delete -m $mapnum -bandwidth 100 $source$path $target$path > /dev/null 2>&1 &    

#大于100G小于1T map数300
    elif [ $size -ge 102400000000 -a $size -lt 1024000000000 ];then
      mapnum=300
    nohup    hadoop distcp  -Dmapreduce.job.name=${taskname} -Dmapreduce.reduce.memory.mb=4096 -Dfs.defaultFS=hdfs://HDFS84008  -Dyarn.app.mapreduce.am.resource.mb=8192 -prbugpt -i -strategy dynamic -skipcrccheck -numListstatusThreads 30 -skipcrccheck -update -delete -m $mapnum -bandwidth 100 $source$path $target$path > /dev/null 2>&1 & 
 #大于1T map数 400   
    else
       mapnum=400
     nohup    hadoop distcp  -Dmapreduce.job.name=${taskname} -Dmapreduce.reduce.memory.mb=4096 -Dfs.defaultFS=hdfs://HDFS84008  -Dyarn.app.mapreduce.am.resource.mb=8192 -prbugpt -i -strategy dynamic -skipcrccheck -numListstatusThreads 30 -skipcrccheck -update -delete -m $mapnum -bandwidth 100 $source$path $target$path > /dev/null 2>&1 & 


    fi  

sleep 0.1


      
   #cd "$(dirname "$0")"
    #str=`date +%Y%m%d%H%M%S`
done

wait



```

可以将校验工具和批量拷贝工具结合使用:
```
#!/bin/bash

  

source /etc/profile

export HADOOP_USER_NAME=xx_effect

export HADOOP_USER_PASSWORD=tx_xxxxxx

  
  

function log {

local logLevel=$1

local message=$2

time='['$(date +"%Y-%m-%d %H:%M:%S")']'

echo "${time} ${logLevel} ${message}" | tee -a ${logFile}

}

  

function logInfo {

log INFO "$*"

}

  

function logError {

log ERROR "$*"

}

  

function wrapFunctionLog {

local command="$*"

logInfo "start function: ${command}"

eval ${command}

local result=$?

if [ ${result} = 0 ]; then

logInfo "success end command: ${command}"

else

logError "fail exec command: ${command}"

fi

}

  
  
  

rootDir=$(

cd "$(dirname "$0")"

pwd

)

  

cd ${rootDir}

  

logTime="$(date +"%Y_%m_%d_%H_%M_%S")"

  

logFile="${rootDir}/xxxxx_diff${logTime}.log"

  

logInfo "start to compare..."

  

startTime="$(date +"%Y-%m-%d %H:%M:%S")"

java -cp /usr/local/service/hadoop/etc/hadoop:/usr/local/service/hadoop/:/usr/local/service/hadoop/share/hadoop/common/*:/usr/local/service/hadoop/share/hadoop/common/lib/*:/usr/local/service/hadoop/share/hadoop/hdfs/*:/usr/local/service/hadoop/share/hadoop/hdfs/lib/*:/usr/local/service/hadoop/share/hadoop/mapreduce/*:/usr/local/service/hadoop/share/hadoop/mapreduce/lib/*:/usr/local/service/hadoop/share/hadoop/yarn/*:/usr/local/service/hadoop/share/hadoop/yarn/lib/*:/data/qy/ca_alg/gary/diff-tool-1.0.jar com.tencent.hdfs.DiffTool 2021-09-01 16:00:00 50 hdfs://nn03.pkx01.rack.xxxxx.com:xxxx hdfs://HDFS84008 /user/ca_alg/data/zrec/models/zplus_kd/nn/kd_base/models >> ${logFile} 2>&1

  

#java -cp /usr/local/service/hadoop/etc/hadoop:/usr/local/service/hadoop/:/usr/local/service/hadoop/share/hadoop/common/*:/usr/local/service/hadoop/share/hadoop/common/lib/*:/usr/local/service/hadoop/share/hadoop/hdfs/*:/usr/local/service/hadoop/share/hadoop/hdfs/lib/*:/usr/local/service/hadoop/share/hadoop/mapreduce/*:/usr/local/service/hadoop/share/hadoop/mapreduce/lib/*:/usr/local/service/hadoop/share/hadoop/yarn/*:/usr/local/service/hadoop/share/hadoop/yarn/lib/*:/data/qy/prod/ca_alg/diff/diff-tool-1.0.jar com.tencent.hdfs.DiffTool ${startTime} 20 hdfs://nn03.pkx01.rack.xxxxx.com:xxxx hdfs://HDFS84008 /user/ca_alg/QuasiMonteCarlo_1618910870021_670540645 /user/ca_alg/QuasiMonteCarlo_1618913321671_1614066023 /user/ca_alg/user >> ${logFile} 2>&1

  

logInfo "end to compare..."

  

dirName="${logTime}"

  

rm -rf ${rootDir}/${dirName}

mkdir ${rootDir}/${dirName}

mv ${rootDir}/xxxxx_qy_*.txt ${rootDir}/${dirName}

  

#logInfo "start to distcp for add..."

#sh ${rootDir}/autodistcp.sh ${rootDir}/${dirName}/xxxxx_qy_need_to_add_forDistcp.txt 2>&1

  

#logInfo "start to distcp for replace..."

#sh ${rootDir}/autodistcp.sh ${rootDir}/${dirName}/xxxxx_qy_need_to_replace_forDistcp.txt 2>&1

#logInfo "end to distcp for replace..."

```

