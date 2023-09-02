

hive  中任务的 map 数最终是由 inputformat 来最终确认的,inputformat 是一个接口,不同的 inputformat 有不同的实现逻辑, 在 MapReduce 框架中，一个 split 就意味着需要一个 Map Task.即 `getSplits` 方法的实现确定了最终的 splits 数,也即 map 数.

```
public interface InputFormat<K,V>{
	public InputSplit[] getSplits(JobConf job,int numSplits) throws IOException; 
	public RecordReader<K,V> getRecordReader(InputSplit split,JobConf job,Reporter reporter) throws IOException; 
}
```



比如说最经典的格式  ` org.apache.hadoop.mapred.TextInputFormat `

由于 TextInputFormat 继承自  FileInputFormat ,FileInputFormat 的实现逻辑中先确定了每一个 split 的大小,然后按照 split size 来对表路径下的文件来做切分.FileInputFormat 的 splitsize 的计算方法为:
`splitSize = max{minSize,min{goalSize,blockSize}} `

- minSize 由 mapreduce.input.fileinputformat.split.minsize 确定
- goalsize = totalSize/numSplits  numSplits  即为任务指定的 mapreduce.job.maps
- blockSize 即为 HDFS 中的文件存储块block的大小



在 hive  sql 任务中,任务会事先获取输入的 inputformat ,即属性 `hive.input.format`  ,这个值默认是 `org.apache.hadoop.hive.ql.io.CombineHiveInputFormat`

CombineHiveInputFormat 在划分Split时，首先挑出不能合并到一起的目录——比如开启了事务功能的路径。这些不能合并的目录必须单独处理，剩下的路径交给私有方法getCombineSplits，这样Hive的一个map task最多可以处理多个目录下的文件


CombineHiveInputFormat 的 getSplits 方法最终实现在 `org.apache.hadoop.mapreduce.lib.input.CombineFileInputFormat` 中

也是先计算每一个 split size 大小,这个大小如下几个参数有关:
- dfs.blocksize
- mapreduce.input.fileinputformat.split.maxsize
- mapreduce.input.fileinputformat.split.minsize.per.rack
- mapreduce.input.fileinputformat.split.minsize.per.rack
在实践中,我们一般只需通过控制 `mapreduce.input.fileinputformat.split.maxsize`  变量来控制分片的结果


---
[Hive中如何确定map数-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/26262#slide-4)
[Hadoop深入学习：InputFormat组件 - 飞翔的荷兰人 - ITeye博客](https://www.iteye.com/blog/flyingdutchman-1876400)
[浅析Hive/Spark SQL读文件时的输入任务划分_Apache Spark中国社区的博客-CSDN博客](https://blog.csdn.net/weixin_45906054/article/details/104489560)



