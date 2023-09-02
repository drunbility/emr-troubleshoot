block是HDFS**操作**文件的最小单位, 它由chunk为单位**组成**, 但由packet为单位进行**传输**

-   1个block由多个chunk data组成, 每个chunk data里存储的都是二进制数据
-   1个block对应有一个`meta`文件, 存放与之对应的chunk checksum
-   HDFS组件间传输数据最小以packet为单位, 1个packet由多个chunk构成, 传输后取出其中的chunk (data+checksum), 最后构成block和`meta`



