2020-02-21 学习一下hdfs
# 1、hdfs搭建
hdfs，分布式文件存储、廉价存储、高吞吐量、容错，大量文件、大文件，tb、pb级存储。

https://blog.csdn.net/sjmz30071360/article/details/79877846   ---什么是hdfs


https://hadoop.apache.org/docs/r1.0.4/cn/hdfs_design.html  ---官网文档

https://blog.csdn.net/apriaaaa/article/details/93207271   ---hdfs操作指令，跟linux一般命令操作很像。

https://www.jianshu.com/p/58b39974abb7。--跟下面的是同一个，下面的是转载的，我参考下面的搭建的。
https://blog.csdn.net/qq_35246620/article/details/88576800
https://blog.csdn.net/wanbf123/article/details/81948026 ---这个用了zookeeper搭建的hdfs
===========
# 2、那什么是hadoop呢？
hadoop中有3个核心组件：

分布式文件系统：HDFS —— 实现将文件分布式存储在很多的服务器上

分布式运算编程框架：MAPREDUCE —— 实现在很多机器上分布式并行运算

分布式资源调度平台：YARN —— 帮用户调度大量的mapreduce程序，并合理分配运算资源

开启hadoop有两种方法：

1、start-all.sh
2、先start-dfs.sh,再start-yarn.sh
————————————————
版权声明：本文为CSDN博主「RobertDowneyLm」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/RobertDowneyLm/article/details/80274265

# 3、hive和mysql的关系？
一直都有一个误区

mysql中存的是hive的数据

实际上mysql存的是hive的元数据 ：数据库，表格名，数据地址等等

1.Hive不存储数据，Hive需要分析计算的数据，以及计算结果后的数据实际存储在分布式系统上，如HDFS上。

2.Hive某种程度来说也不进行数据计算，只是个解释器，只是将用户需要对数据处理的逻辑，通过SQL编程提交后解释成MapReduce程序，然后将这个MR程序提交给Yarn进行调度执行。所以实际进行分布式运算的是MapReduce程序

3.因为Hive为了能操作HDFS上的数据集，那么他需要知道数据的切分格式，如行列分隔符，存储类型，是否压缩，数据的存储地址等信息。为了方便以后操作所以他需要将这些信息通过一张表存储起来，然后将这张表（元数据）存储到mysql中。为了啥存储到mysql里（实际是远程mysql）,因为hive本身就是一个解释器，所以他不存储数据 。

资料连接：https://blog.csdn.net/qq_26442553/article/details/80206562 

