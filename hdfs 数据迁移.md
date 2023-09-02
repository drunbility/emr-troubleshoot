### Distcp 工具同步数据

  HDFS数据迁移可以通过Hadoop社区标准的[DistCp工具](http://hadoop.apache.org/docs/r3.2.4/hadoop-distcp/DistCp.html)迁移，可以实现全量和增量的数据迁移。为减轻现有集群资源压力，建议在新旧集群网络连通后在新集群执行**distcp**命令。

  - 全量数据同步
  `hadoop distcp -pbugpcax -m 4000 -bandwidth 100 hdfs://oldclusterip:8020/user/hive/warehouse /user/hive/warehouse`

  - 增量数据同步
  `hadoop distcp -pbugpcax -m 4000 -bandwidth 100  -update –delete hdfs://oldclusterip:8020/user/hive/warehouse /user/hive/warehouse`

  参数说明:

  - `-p`：preserver 设置拷贝后保留源文件哪些属性. b:块大小,u:用户,g:用户组,p:读写权限,r:副本数, c:check sum,a:acl,x: xattr,t: 时间戳,q: quota 信息. 例如:`-pbugprcaxtq` 保留源文件所有属性
  - `-m`：指定map数，和集群规模、数据量有关。例如集群有2000核CPU，就可以指定2000个map。
  - `-bandwidth`：指定单个map的同步速度，是靠控制副本复制速度实现的，是大概值。
  - `-update`：校验源文件和目标文件的checksum和文件大小，如果不一致源文件会更新掉目标集群数据，新旧集群同步期间还有数据写入，可以通过`-update`做增量数据同步。
  - `-delete`：如果源集群数据不存在，新集群的数据也会被删掉。
  - `oldclusterip`：填写旧集群namenode ip，多个namenode情况填写当前状态为active的。
  - -strategy=dynamic: 动态 Map 任务分配,加快复制


对于大量的数据比如数十 PB ,应该把大目录拆分成[[distcp 拷贝脚本|子目录来拷贝]]
  

### HDFS权限配置

HDFS有权限设置，确定旧集群是否有ACL规则，是否要同步，检查新旧集群**dfs.permissions.enabled**和**dfs.namenode.acls.enabled**的配置是否一致，按照实际需要修改。

如果有ACL规则要同步，**distcp**参数后要加`-p`同步权限参数。如果distcp操作提示xx集群不支持 ACL，说明对应集群没配置ACL规则。新集群没配置ACL规则可以修改配置并重启namenode。旧集群不支持，说明旧集群根本就没有ACL方面的设置，也不需要同步。


### 迁移后数据校验

一般 distcp 拷贝完成时会同时做 [[checksum  校验和]],如果需要事后再次做校验的话,可以使用 hdfs-tool 工具:

HDFS Diff & Checksum v1.0 工具（以下简称difftool） 是用于对HDFS同步后的数据进行diff 和 checksum 对比校验的工具。


其功能是将两个HDFS文件夹进行对比，这两个文件夹可以分别定义为 source 和 target。具体功能描述如下：

首先分别递归扫描source 和 target，对于同级目录进行对比分类,最终对 source 和 target 生成如下结论:


***工具旧版本截图中七区代表 target ,四区代表 source,新版本中的输出已经纠正***


![[Pasted image 20221228151737.png]]

使用方法:

将工具包解压到 emr 某机器某个目录,例如: /data/qy/file_diff ,该目录下有 三个文件:diff-tool-2.0.jar,difftool.properties,log4j.properties

配置 difftool.properties:

![[Pasted image 20221228151759.png]]

然后执行:

`java -Xmx128g -Xms128g -cp /usr/local/service/hadoop/etc/hadoop:/usr/local/service/hadoop/:/usr/local/service/hadoop/share/hadoop/common/*:/usr/local/service/hadoop/share/hadoop/common/lib/*:/usr/local/service/hadoop/share/hadoop/hdfs/*:/usr/local/service/hadoop/share/hadoop/hdfs/lib/*:/usr/local/service/hadoop/share/hadoop/mapreduce/*:/usr/local/service/hadoop/share/hadoop/mapreduce/lib/*:/usr/local/service/hadoop/share/hadoop/yarn/*:/usr/local/service/hadoop/share/hadoop/yarn/lib/*:/data/qy/file_diff:/data/qy/file_diff/diff-tool-2.0.jar -Dlog4j.configuration=file:///data/qy/file_diff/log4j.properties com.tencent.hdfs.DiffTool >> xxxx.log 2>&1`


