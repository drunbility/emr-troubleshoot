#### 现象
用户反馈他们对 hive 的一些表分区数非常多(几万以上)的表进行 alter table add columns 加字段时,会非常慢,达到几分钟.客户定位到当执行 alter 语句时, hive meta db 元数据库的读取负载很高.

####  背景说明及说明

-   cascade 关键字  
	hive 1.1 版本后,表元数据和表的分区元数据是分开处理的.alter table 语句后面增加了两个关键字来控制元数据修改行为: cascade 和 restrict .restrict 是默认值,指的是 alter table 仅修改表元数据. 如果声明 cascade 则意味着修改表元数据的同时会同步修改已有表分区的元数据.举例说明:比如一个表 test 有一个分区 dt=1 ,如果对这个表执行 alter table test add columns(col1 string); 那么仅对表新增字段 col1, 表分区 dt=1 这个分区内没有新增 col1 字段,导致已有的这个分区内的 col1 字段无法插入值(但后续新增的分区会正常使用).如果执行时加上了 cascade 关键字:  alter table test add columns(col1 string) cascade; 那么已有的分区元数据会同步更新,col1 字段能正常插入值.
-   回到客户反馈的慢问题,通过测试复现,查看 cdb 的慢查询,定位到在新增字段的时候 ,hive 去元数据库里查询了表的所有分区信息,当表的分区数达到上万,这个查询语句就非常慢了.
![[Pasted image 20230216200441.png]]
-   现在的疑问点有两个,一是在现在的版本 3.1 中,执行 add columns 时没有加上 cascade 关键字,为什么依然要去获取所有的表分区信息.二是客户反馈在客户自建的集群 hive 1.2 版本中,同样的语句执行会比较快.当时猜测可能 hive 在新版本中增加了某项 feature ,会需要获取所有分区信息这个逻辑.要解决这两个疑问,需要从 hive 内核的层面去定位具体逻辑.

#### hive2 和 hive3 的差异处

最终定位到造成 1.2 和 3.1 的执行快慢差异的关键原因在于 hive 3.0 中修复了一个 bug : [[HIVE-18138] Fix columnstats problem in case schema evolution - ASF JIRA (apache.org)](https://issues.apache.org/jira/browse/HIVE-18138)  .

看代码修改记录, hive 3.0 前,alter table 会首先判断 cascade 关键字,才会进入 update 表分区元数据逻辑. 这个 PR 以后, 无论加不加 cascade  都会去把存在的partition做一次update动作. 这里其实是一个 trade off ,为了保证 columnstats 统计信息的正确性,需要做这么一次更新操作

![[Pasted image 20230216200542.png]]


#### 修复

社区好像没有修复这个问题,于是直接在这块逻辑上面做修复,在 316 到 320 行增加逻辑判断:
上面 HIVE-18138 jira的patch,会引入一个新的问题,就是不管统计数据有没有变化,都会调用mdb.alterPartition,在分区很多的情况下,对metastore的压力比较大,这里只需要判断统计数据没有变化,就不要alterPartition就好了

修改如上:
```

---
 .../hive/metastore/HiveAlterHandler.java      | 20 ++++++++++++++++---
 1 file changed, 17 insertions(+), 3 deletions(-)

diff --git a/standalone-metastore/src/main/java/org/apache/hadoop/hive/metastore/HiveAlterHandler.java b/standalone-metastore/src/main/java/org/apache/hadoop/hive/metastore/HiveAlterHandler.java
index f328ad1815..479ee430b9 100644
--- a/standalone-metastore/src/main/java/org/apache/hadoop/hive/metastore/HiveAlterHandler.java
+++ b/standalone-metastore/src/main/java/org/apache/hadoop/hive/metastore/HiveAlterHandler.java
@@ -329,9 +329,11 @@ public void alterTable(RawStore msdb, Warehouse wh, String catName, String dbnam
               if (cascade) {
                 msdb.alterPartition(catName, dbname, name, part.getValues(), part);
               } else {
-                // update changed properties (stats)
-                oldPart.setParameters(part.getParameters());
-                msdb.alterPartition(catName, dbname, name, part.getValues(), oldPart);
+                if (!isPartitionStatsEquals(part, oldPart)) {
+                  // update changed properties (stats)
+                  oldPart.setParameters(part.getParameters());
+                  msdb.alterPartition(catName, dbname, name, part.getValues(), oldPart);
+                }
               }
             }
             msdb.alterTable(catName, dbname, name, newt);
@@ -398,6 +400,18 @@ public void alterTable(RawStore msdb, Warehouse wh, String catName, String dbnam
     }
   }
 
+  private boolean isPartitionStatsEquals(Partition part, Partition oldPart) {
+    String stats = null;
+    String oldStats = null;
+    if (part != null && part.getParameters() != null) {
+      stats = part.getParameters().get(StatsSetupConst.COLUMN_STATS_ACCURATE);
+    }
+    if (oldPart != null && oldPart.getParameters() != null) {
+      oldStats = oldPart.getParameters().get(StatsSetupConst.COLUMN_STATS_ACCURATE);
+    }
+    return (stats == null && oldStats == null) || (stats != null && stats.equals(oldStats));
+  }
+
   /**
    * MetaException that encapsulates error message from RemoteException from hadoop RPC which wrap
    * the stack trace into e.getMessage() which makes logs/stack traces confusing.
-- 
2.31.1


```


