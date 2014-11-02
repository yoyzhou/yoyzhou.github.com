---
layout: post
title: "Hadoop in Action学习笔记"
date: 2013-04-14 15:41
comments: true
categories: [Hadoop, MapReduce, 读书笔记, Bigdata]
keywords: Hadoop, MapReduce, 读书笔记, Bigdata, 大数据, ReadingNote, Hive, HBase, Pig
description: Hadoop实践（Hadoop in Action）读书笔记
---


####第一章 Hadoop简介

现今，互联网每天都产生海量的数据，现有工具对于TB、PB级别大规模分布式海量数据变得无力处理。

Google首先推出了处理大规模分布式数据的MapReduce计算范式，Doug Cutting领导开发了一个开源版的MapReduce，后来成为Hadoop。

#####什么是Hadoop

Hadoop是一个开源框架，可编写和运行分布式应用处理大规模数据。一般认为Hadoop包含两个核心部分：HDFS，Hadoop分布式文件系统和MapReduce，一种分布式编程范式（分布式存储和分布式计算）。

	**优点：**
	• 方便
	• 健壮
	• 线性可扩展
	• 简单

#####分布式系统  VS 大型单机服务器

	向外扩展 VS  向上扩展

	构建在一般/低端商用机器上，廉价

	高端服务器，价格昂贵

#####Hadoop设计理念和SETI@Home的区别

Hadoop设计用于数据密集型任务，遵循将程序向数据移动的设计哲学，即尽可能的保持数据不动，减少I/O的访问，程序文件的数量级相对于数据的数量级小得多

SETI@Home设计用于计算密集型任务，遵循将数据向计算资源（计算机）移动的设计哲学；因为数据传输量小，但是需要大量的计算资源（CPU时间）

#####Hadoop与SQL数据库的比较
都是处理数据，SQL数据库用于处理结构化数据

Hadoop应用针对的是文本数据/可能是结构的也可能使非结构或者半结构化数据

1\. 向外扩展 代替向上扩展

2\. 用键/值对 代替关系表

3\. 用函数式编程（MapReduce）代替申明式查询（SQL）

4\. 用离线处理代替在线处理

数据密集型分布式应用中，数据传输的代价昂贵，应保持数据存储和处理紧密的绑定在一起。

#####Hadoop的历史
2004年 Goole发表Google文件系统（GFS）和MapReduce框架

Doug Cutting将Nutch项目移植到GFS和MapReduce上，并设立了专门的项目充实这两种技术，于是就有了Hadoop

2006年 Yahoo聘用Doug Cutting，让他和一个专门团队一起改进Hadoop

2008年 Hahoop成为Apache的顶级项目

####第二章 初识Hadoop

**"运行Hadoop"意味着在不同服务器运行一组守护进程（daemons）:**

	0 NameNode (名字节点)
	1 DataNode (数据节点)
	2 Secondary NameNode (次名字节点)
	3 JobTracker (作业跟踪节点)
	4 TaskTracker (任务跟踪节点)

Hadoop在分布式计算和分布式存储中都采用了**主/从(Master/Slaver)**的结构

NameNode，JobTracker是Master节点

DataNode，TaskTracker和Slaver节点

#####运行Hadoop 
配置文件：core-site.xml, hdfs-site.xml和mapred-site.xml

conf/masters - master节点主机列表

conf/slaves - slave节点主机列表

运行的三种模式：本地（单机）模式，伪分布模式和全分布模式

	bin/hadoop namenode -format
	bin/start-all.sh
	jps


#####基于WEB的管理界面
NameNode状态界面： namenode-host:50070

JobTracker状态界面： jobtacker-host:50030

####第三章 Hadoop组件

**HDFS文件系统**

HDFS是一种文件系统，专为MapReduce这类框架下的大规模分布式处理而设计的

可以处理单个的大数据集（比如100TB），而大多数文件系统物理实现这点。

**？**是不是说HDFS处理单个大数据集更加有效，而处理小文件的大量数据变现的不是很有优势，当然，如果这样也可以先把小文件组成大文件。

**？**文件要多大才能适合HDFS/Hadoop的处理呢?

可不要为了数据大而大，数据应该在必要的清洗和预处理

#####Hadoop的基本文件命令

hadoop fs -cmd \<args\>

HDFS中，默认的根目录是/user/$USER 其中USER是你登录系统的用户名

1\. 添加文件和目录

`hadoop fs -mkdir <dir-name>`

将在/user/$USER目录下创建\<dir-name\>，如果\<dir-name\>包含多层目录且上层目录不存在，hadoop fs -mkdir会创建指定的多层目录结构。

如:

`hadoop fs -mkdir inputs/dataset1 会创建/user/$USER/inputs/dataset1`

2\. 列举文件清单

`hadoop fs -ls <file-dir-name>`

和Unix系统ls本地文件类似

`hadoop fs -lsr <file-dir-name>`

查看所有文件和文件的子目录，递归的列出文件清单

3\. 复制本地文件到HDFS 

`hadoop fs -put <file-dir-name-to-put-in-hdfs> <file-dir-on-hdfs>`

4\. 从HDFS上检索文件到本地系统

`hadoop fs -get <file-dir-on-hdfs-to-get> <file-dir-name-on-localhost>`

这是一个与hadoop fs -put相反的操作。

5\. 查看HDFS上文件的内容

`hadoop fs -cat <file-name-to-cat>`

例如：

`hadoop fs -cat example.txt`

同样假设文件在默认的工作目录/user/$USER下

6\. 删除文件/目录

`hadoop fs -rm <file-name-to-remove>`

例如：

`hadoop fs -rm example.txt`

这个命令用来删除文件，要删除文件夹是使用 -rmr 命令：

`hadoop fs -rmr <dir-name-to-remove>`

例如：

`hadoop fs -rmr input`

这里的`r`代表递归的删除文件。

7\. 查看文件命令帮助

`hadoop fs -help <cmd>`

例如：

`hadoop fs -help rmr`

查看rmr命令的帮助文件

8\. hadoop文件命令与Unix管道一起使用

例如：

`hadoop fs -cat example.txt | head -n 20`

#####Hadoop数据的读和写

遵循并行处理数据分片原则的输入数据通常为单一的大文件。这也是Hadoop分布式文件系统的设计策略。

FSDataInputStream支持随机读取，这一特性涉及到MapReduce的数据分片机制。

随机读取，当一个任务失败时，恢复时不用从头读取文件，只需要从失败的位置进行恢复。

**InputFormat**

Hadoop分割与读取输入文件的方式在InputFormat接口中定义，TextInputFormat是InputFormat的默认实现。Hadoop提供的其他的InputFormat实现有：

KeyValueTextInputFormat，SequenceFileInputFormat和NLineInputFormat

**OutputFormat**

通过使用IntWritable类变量可以提高reduce()的性能，IntWritable类的处理性能要比Text高。

####第四章 编写MapReduce基础程序

**mapper container partitioner reducer** 

#####使用Combiner提升性能

Combiner作用上与Reduce等价，起到聚合、结合作用，原则上程序必须没有Combiner也能够得到正确的结果

1\. 网络洗牌时减少数据传输流量，考虑10亿条Mapper键值对的输出，如果没有Combiner会在网络洗牌过程中造成多大的流量

2\. 数据分布不均匀时，造成大量Mapper的键值对输出跑向同一个Reducer

原理就是通过Combiner减少Mapper的输出数量以降低网络和Reducer上的压力，Combiner位于Mapper和Reducer中间，被视为是Reducer的助手，在数据转换上必须与Reducer等价，也就是如果我们去掉Combiner，Reducer的输出应该保持不变。

但是Combiner未必会提高性能，这要看Combiner是否能有效的减少Mapper输出记录的数量。

####第五章 高阶MapReduce

#####MapReduce之间的依赖

Hadoop通过Job和JobControl类来管理作业之间的非线性依赖关系。

x.addDependingJob(y)

ChainMapper ChainReducer

链接MapReduce是，mapper、reducer之间的输入输出按照值传递还是引用传递的问题。P100

如果确保上游的Map/Reduce不是用其输出数据，或者下游Map/Reduce不改变上游的输出数据，可以使用by reference以提高一定的性能

#####联结不同来源的数据

1\. Reduce侧的联结

repartitioned join 重分区联结

repartitioned sort-mergejoin 重分区排序-合并联结

结合DataJoin包使用钩子处理数据流P93

2\. Mapper侧的联结P98

**DistributedCache 分布式缓存**

应用场景：一个数据源较大，另一个数据源可能小几个数量级，将较小的数据源装入内存，复制到所有的Mapper，在map阶段执行联结。通常小的数据源也叫做“背景数据”

####第六章 编程实践

#####本地模式

**1 计算的完整性检查**

数学和逻辑错误在数据密集型程序中更加普遍，而它们往往并不明显！

数学计数或者算术如何检查正确性，在分布式计算中数学公式可能要重新设计，但是如何验证计算的完整性和准确性也就十分的重要。

**方法：**

可是通过宏观的查看一些指标，比如平均值，整体计数、最大值等来进行初步验证

**2 回归测试**

可能代价会很大，但是可以选取一部分数据集进行测试。所谓回归测试就是在同一数据集上比较代码修改前后算法的输出结果是否一致，如果每次回归测试的结果都是一样的，那么可以判定修改没有导致出现新的问题。

**3 考虑使用LONG而不是INT**

#####伪分布模式
1 日志

2 JOBTRACKER的WEB管理界面

3 杀掉作业

#####生产集群上的监控与调试
计数器 Reporter.incrCounter()

跳过坏记录 skipping模式的设置P126

用IsolationRunner重新运行出错的任务

#####性能调优

**Hadoop的线性可扩展性**

######1 通过combiner来减少网络流量

######2 减少输入数据
	a. 采样数据，只处理数据的子集
	b. “重构”数据，仅使用需要的字段（要求开发人员清楚的了解数据和自己的业务需求）
	#上面两个方案一个是**横切**一个是**纵切**，都是减少输入数据的方法

######3 使用压缩

即使使用了combiner，在map阶段的输出也可能很大。这些中间数据必须被存储在磁盘上，并在网络上重排。压缩这些中间数据会提高大多数MapReduce作业的性能。Hadoop内置支持压缩和解压缩。

	压缩的参数配置P130
	mapred.compress.map.output
	mapred.map.output.compression.codec
	SequenceFIleOutputFormat 序列文件输出

######4 重用JVM

mapred.job.reuse.jvm.num.task 指定一个JVM可以运行的最大任务数

######5 根据猜测执行来运行

在所有的mapper完成之前，reducer都不会启动；类似的，在所有的reducer完成之前，一个作业也不会结束。

**问题：当其中一个任务变慢时（注意不是失效），MapReduce如何处理？**

Hadoop会注意到运行速度缓慢的任务，并安排在另一个节点上并行执行相同的任务。

	**参数：**

	mapred.map.tasks.speculative.execution
	mapred.reduce.tasks.speculative.execution

######6 代码重构与算法重写

6.1 将Streaming程序重写为Java程序

6.2 在一次作业中集中一次处理类似的任务，而不是每一个任务都启动一个作业，比如计算最大值、最小值和均值等在同一数据集上的计算可以在一个Job中完成，而不要分为不同的Job

6.3 设计特定的新的适合MapReduce框架的算法
 
####第七章 细则手册

#####7.1 向任务传递作业定制的参数

在JobConf中定制自己的参数，在所有的Task中都能读取到。

#####7.2 探查任务特定信息

#####7.3 划分为多个输出文件
MultipleOutputFormat 键/值区分，写入不同的文件， 一个Collector 不同文件名

MultipleOutputs	属性区分，写入不同的文件，多个Collector写文件

#####7.4 以数据库作为输入输出
**DBOutputFormat**

#####7.5 保持输出的顺序


####第八章 管理Hadoop

#####8.1 为实际应用设置特定参数值

#####8.2 系统体检
	bin/hadoop fsck <path> #file system check
	bin/hadoop dfsadmin -report

#####8.3 权限设置

#####8.4 配额管理
	bin/hadoop dfsadmin -setQuota <N> dir
	bin/hadoop dfsadmin -setSpaceQuota <N> dir
	bin/hadoop fs -count -q dir #显示目录的配额
 
#####8.5 启动回收站

fs.trash.interval属性（以分钟为单位）

#####8.6 删减DataNode

	dfs.hosts.exclude=file-point-to-exclude-host-list
	bin/hadoop dfsadmin -refreshNodes

#####8.7 增加DataNode

#####8.8 管理NameNode和SNN
减轻NameNode负担的一种方法是增加数据块的大小，以降低文件系统中元数据的数据量，dfs.block.size 默认64M

######SNN是做什么的？
取了一个不妥的名字。首先并不是NameNode的失效备份

把SNN视为一个检查点服务器更为合适，用于合并下面两个文件以形成系统快照。

	可能在0.21中被弃用
	FsImage
	EditLog

#####8.9 恢复失效的NameNode
备份NameNode的元数据文件（dfs.name.dir）到SNN节点，或者其他多个节点。

#####8.10 感知网络布局和机架的设计
**DataNode数据块的副本策略**
标准副本为3个：

第1个 在同一个DataNode上

第2个 不同的机架上的DataNode上

第3个 与第二个同机架的不同DataNode上

如果大于3个副本，其他的随机放置的不同节点上

topology.script.file.name

机架感知，配置Hadoop是NameNode知道DataNode的机架分布结构


#####8.11 多用户作业调度
8.11.1 多个JobTracker

8.11.2 公平调度器 （Facebook）

####第九章 在云上运行Hadoop
**租用 经济**
Amazon Web Services(AWS)

Elastic Compute Clous(EC2) 弹性计算云

Simple Storage Service(S3)简单存储服务

1 AWS与外部网络通信的带宽需要收取费用，EC2内部不需要收费。

2 先存储在S3 在从S3中复制到主节点

S3中存储数据同样要收费 但是 存储空间可扩展、一次存储可以多次使用、资费相对低、S3和EC2是内部网，之间复制数据没有费用且速度快

3 还可以直接将S3作为输入输出文件地址，而不用复制数据到HDFS

####第十章 用Pig编程（Not Finished）
Pig是架构在Hadoop之上的高级数据处理层。

**易用性、高性能、大规模可扩展能力**是所有Hadoop子项目的期望

**“Pig吃任何东西”**

Pig Latin命令运行： Grunt Shell、脚本文件、嵌入在Java程序中。


####第十一章 Hive及Hadoop群
Pig——一种高级数据流语言

Hive——一种类SQL数据仓库基础设施

HBase——一种模仿Google BigTable的分布式的、面向列的数据库

ZooKeeper——一种用于管理分布式应用之间共享状态的可靠的协同系统

Cascading——是Hadoop上用于组装和执行复杂数据处理工作流的一个API

Hama——矩阵计算软件包，用于计算乘积、逆、特征值、特征向量和其他矩阵运算

Mahout——基于Hadoop实现机器学习算法



##### Hive
Hive是建立在Haddop基础之上的数据仓库软件包

HiveQL——类SQL的数据查询语言

MetaStore ——用于存放元数据的组件，存储在Derby关系数据库中（存取速度的考虑）

`---EOF---`



