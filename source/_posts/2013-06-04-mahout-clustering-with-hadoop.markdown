---
layout: post
title: "在Hadoop上运行Mahout KMeans聚类分析"
date: 2013-06-04 20:18
comments: true
categories: [Mahout, Hadoop, Clustering, Data Mining]
keywords: Mahout, Hadoop, Clustering, Data Mining, Machine Learning, Java, MapReduce
description: 本文首先介绍了如何配置Mahout的Hadoop的运行环境，然后介绍如何使用mahout kmeans命令行将聚类分析运行在Hadoop上。
---

上一篇文章[“Mahout与聚类分析”][last_post]介绍了如何使用Mahout进行聚类分析的步骤，并且结合实例使用K-Means对微博名人共同关注数据进行了共被关注聚类分析。Mahout运行有本地运行和Hadoop运行两种模式，本地运行是指在用户本地的单机模式下运行，就像运行其他普通的程序一样，但是这样这样就不能最大限度的发挥出Mahout的优势，在本文中我们介绍如何让我们的Mahout聚类分析程序在Hahoop集群上运行（在实际操作中笔者使用的伪分布Hadoop，而不是真正的Hadoop集群）。

####配置Mahout运行环境

Mahout运行配置可以在`$MAHOUT_HOME/bin/mahout`里面进行设置，实际上`$MAHOUT_HOME/bin/mahout`就是Mahout在命令行的启动脚本，这一点与Hadoop相似，但也又不同，Hadoop在$HADOOP\_HOME\conf下面还提供了专门的hadoop-env.sh文件进行相关环境变量的配置，而Mahout在conf目录下没有提供这样的文件。

#####MAHOUT_LOCAL与HADOOP_CONF_DIR

以上的连个参数是控制Mahout是在本地运行还是在Hadoop上运行的关键。

`$MAHOUT_HOME/bin/mahout`文件指出，**只要设置`MAHOUT_LOCAL`的值为一个非空（not empty string）值，则不管用户有没有设置HADOOP_CONF_DIR和HADOOP_HOME这两个参数，Mahout都以本地模式运行**；换句话说，如果要想Mahout运行在Hadoop上，则MAHOUT_LOCAL必须为空。 

`HADOOP_CONF_DIR`参数指定Mahout运行Hadoop模式时使用的Hadoop配置信息，这个文件目录一般指向的是$HADOOP_HOME目录下的conf目录。

除此之外，我们还应该设置`JAVA_HOME`或者`MAHOUT_JAVA_HOME`变量，以及必须将Hadoop的执行文件加入到PATH中。

综上所述：

1\. 添加`JAVA_HOME`变量，可以在直接设置在`$MAHOUT_HOME/bin/mahout`中，也可以在user/bash profile里面设置(如`./bashrc`)

2\. 设置MAHOUT_HOME并添加Hadoop的执行文件到PATH中

两个步骤在`~/.bashrc`的设置如下：

	export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-i386
	#export HADOOP_HOME=/home/yoyzhou/workspace/hadoop-1.1.2
	export MAHOUT_HOME=/home/yoyzhou/workspace/mahout-0.7
	export PATH=$PATH:/home/yoyzhou/workspace/hadoop-1.1.2/bin:$MAHOUT_HOME/bin

编辑完`~/.bashrc`,重启Terminal即可生效。

3\. 编辑`$MAHOUT_HOME/bin/mahout`，将`HADOOP_CONF_DIR`设置为`$HADOOP_HOME\conf`
	
	HADOOP_CONF_DIR=/home/yoyzhou/workspace/hadoop-1.1.2/conf

读者可以将相关的Hadoop和Mahout主目录修改自己系统上面的目录地址，设置好之后重启Terminal，在命令行输入mahout，如果你看到如下的信息，就说明Mahout的Hadoop运行模式已经配置好了。

	MAHOUT_LOCAL is not set; adding HADOOP_CONF_DIR to classpath. 
	Running on hadoop...

要想使用本地模式运行，只需在`$MAHOUT_HOME/bin/mahout`添加一条设置`MAHOUT_LOCAL`为非空的语句即可。


####Mahout命令行

Mahout为相关的数据挖掘算法提供了相应的命令行入口，同时提供了一些数据分析处理的用到的工具集。这些命令可以通过在终端输入`mahout`获得。以下显示了输入`mahout`的部分信息：
	
	....
	Valid program names are:
	  arff.vector: : Generate Vectors from an ARFF file or directory
	  baumwelch: : Baum-Welch algorithm for unsupervised HMM training
	  canopy: : Canopy clustering
	  cat: : Print a file or resource as the logistic regression models would see it
	  cleansvd: : Cleanup and verification of SVD output
	  clusterdump: : Dump cluster output to text
	  ....
	  fkmeans: : Fuzzy K-means clustering
	  fpg: : Frequent Pattern Growth
	  hmmpredict: : Generate random sequence of observations by given HMM
	  itemsimilarity: : Compute the item-item-similarities for item-based collaborative filtering
	  kmeans: : K-means clustering
	....

####Mahout kmeans

在上一篇[文章][last_post]，我们通过调用KMeansDriver.run()方法从Mahout程序中直接启动KMeans算法，这种方式对于在本地调试程序非常有用，但是在真实项目中，无论是使用Hadoop模式运行，还是本地运行，从命令行运行Mahout的相关算法更加合适，这样的好处是我们只需要给Mahout提供符合相应算法要求的输入数据，即可以利用Mahout分布式处理的优势。比如在本例中，使用kmeans算法，只需要事先将数据处理成Mahout kmeans算法要求的输入数据，然后在命令行调用`mahout kmeans [options]`即可。

在命令行输入不带任何参数的`mahout kmeans`，Mahout将为你列出在命令行使用kmeans算法的使用方法。

	Usage:                                                                          
	 [--input <input> --output <output> --distanceMeasure <distanceMeasure>         
	--clusters <clusters> --numClusters <k> --convergenceDelta <convergenceDelta>   
	--maxIter <maxIter> --overwrite --clustering --method <method>                  
	--outlierThreshold <outlierThreshold> --help --tempDir <tempDir> --startPhase   
	<startPhase> --endPhase <endPhase>]                                             
	--clusters (-c) clusters    The input centroids, as Vectors.  Must be a         
		                        SequenceFile of Writable, Cluster/Canopy.  If k is  
		                        also specified, then a random set of vectors will   
		                        be selected and written out to this path first 

相关的参数我们已经在上篇[文章][last_post]中提到过。

具体的步骤如下：

	1. 将数据处理为Mahout向量（Vector）的形式
	2. 将Mahout向量转化为Hadoop SequenceFile
	3. 创建K个初始质心\[可选\]
	4. 将Mahout向量的SequenceFile复制到HDFS上
	5. 运行`mahout kmeans [options]`

下面的命令显示使用CosineDistanceMeasure对data/vectors目录下Mahout向量数据进行kmeans聚类分析，输出结果保存在output目录下。
 
	mahout kmeans -i data/vectors -o output -c data/clusters \
	-dm org.apache.mahout.common.distance.CosineDistanceMeasure \
	-x 10 -ow -cd 0.001 -cl

更加详细的命令行参数可以在Mahout wiki [k-means-commandline][km_cmd]上查找到。

####总结

本文首先介绍了如何配置Mahout的Hadoop的运行环境，然后介绍如何使用mahout kmeans命令行将聚类分析运行在Hadoop上。


[last_post]:/blog/2013/05/26/clustering-with-mahout/
[km_cmd]:https://cwiki.apache.org/MAHOUT/k-means-commandline.html
