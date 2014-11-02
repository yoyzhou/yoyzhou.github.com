---
layout: post
title: "Mahout与聚类分析"
date: 2013-05-26 21:53
comments: true
categories: [Mahout, Hadoop, Clustering, Data Mining]
keywords: Mahout, Hadoop, Clustering, Data Mining, Machine Learning, Java, MapReduce
description: 本文简要的介绍了Mahout以及聚类分析，并结合实例说明了如何使用Mahout的K-Means算法进行聚类分析。Mahout是可扩展的开源机器学习软件包，目前实现的机器学习算法主要包含有协同过滤/推荐引擎，聚类和分类三个部分。Mahout从开始设计就旨在建立可扩展的机器学习软件包，她的一些部分的实现直接创建在Hadoop之上，这就使得其具有进行大数据处理的能力，也是Mahout最大的优势所在。当你正在研究的数据量大到不能在一台机器上运行时，就可以选择使用Mahout，让你的数据在Hadoop集群的进行分析。
---

本文简要的介绍了Mahout以及聚类分析，并结合实例说明了如何使用Mahout的K-Means算法进行聚类分析。

####Mahout

Mahout是Apache下的开源机器学习软件包，目前实现的机器学习算法主要包含有**协同过滤/推荐引擎**，**聚类**和**分类**三个部分。Mahout从设计开始就旨在建立可扩展的机器学习软件包，用于处理大数据机器学习的问题，当你正在研究的数据量大到不能在一台机器上运行时，就可以选择使用Mahout，让你的数据在Hadoop集群的进行分析。Mahout某些部分的实现直接创建在Hadoop之上，这就使得其具有进行大数据处理的能力，也是Mahout最大的优势所在。相比较于[Weka][wk]，[RapidMiner][rdm]等图形化的机器学习软件，Mahout只提供机器学习的程序包（library），不提供用户图形界面，并且Mahout并不包含所有的机器学习算法实现，这一点可以算得上是她的一个劣势，但前面提到过Mahout并不是“又一个机器学习软件”，而是要成为一个“可扩展的用于处理大数据的机器学习软件”，但是我相信会有越来越多的机器学习算法会在Mahout上面实现。


####聚类分析

物以类聚，人以群分。顾名思义，聚类分析就是将不同的对象分为不同的组或簇，它与我们日常生活所理解的类的概念是相一致的。聚类分析能够帮助我们很好地了解对象之间的类与结构，聚类分析也能够帮助我们将个别对象指派到相应的类。


> 聚类分析仅根据在数据中发现的描述对象及其关系的信息，将数据对象分组。其目标是，组内的对象相互之间是相似的（相关的），而不同组之间的对象是不同的（不相关的）。组内的相似性（同质性）越大，组间差别越大，聚类就越好。<br/>《数据挖掘导论》 Pang-Ning Tan等

常见的聚类分析有K-Means聚类和层次聚类两种。此外还有基于密度的、基于图的以及基于模型的聚类分析方法。

从上面的定义中可以看出，聚类的分析的关键之一在于相似性的度量，常用的相似性有基于距离的相似度，余玄相似度，Jaccard相似度以及Pearson相关系数等。

####Mahout聚类分析

Mahout的聚类分析提供了K-Means聚类、Fuzzy K-Means聚类和基于模型的聚类等方法。下面我们以K-Means聚类来说明如何使用Mahout进行聚类分析。

#####1. 数据准备

在进行数据分析时，第一步必然是准备分析数据。根据研究或者项目的要求收集数据，并且对数据进行必要的预处理。比如我下面我们要提到的微博用户共被关注分析中，首先我们要收集微博用户的关注信息，然后从关注信息中提取出用户共被关注的数据，也就是进行数据预处理。最终我们需要的数据可能就像下面的格式这样直白明确：“用户1,用户2,共被关注次数”。如果你是要进行文档的聚类分析，那么首先需要的获得的是文档的TF-IDF向量（文档-词向量）。

#####2. 将数据转换成Mahout中的Vector

Mahout要求每一个输入数据都应该是一个Vector，并且以Hadoop的SequenceFile类型存储。简单来说N维**Vector**就是一条N维**记录**，其中**维**就是记录的**属性**。比如一个TF-IDF向量就是一个**文档**记录，里面的属性就是**词**，有n个词就有n个维。有时候属性也叫**特征**。

在准备好Vectors之后，需要将这些记录写入到Hadoop Sequence文件中。下面的代码片段演示和如何创建Mahout Vector和将记录写成Sequence文件。

{% codeblock lang:java %}

//...create Vector
//NamedVector is a Vector Wrapper class which owns a name for each vector, very convenient class 
NamedVector vec = new NamedVector(new RandomAccessSparseVector(cflist.size() + 1), account);
Iterator<CoFollowedNode> nodeItr = cflist.iterator();
while(nodeItr.hasNext()){
	CoFollowedNode cfnode= nodeItr.next();
	//set vector's n-dimension with co-followed number
	vec.set(cfnode.getDemension(), cfnode.getCoFollowedNum());
}


/**
  * Write Vectors to Hadoop sequence file, it's required per Mahout implementation that Vectors passed
  * into Mahout clusterings must be in Hadoop sequence file. 
  * 
  **/
private static void writeVectorsToSeqFile(ArrayList<NamedVector> vectors, String filename,
		FileSystem fs, Configuration conf) throws IOException {
		
	Path path = new Path(filename);
	SequenceFile.Writer writer = new SequenceFile.Writer(fs, conf, path,
				Text.class, VectorWritable.class);

		
	VectorWritable vec = new VectorWritable();
		
	//write vectors to file, the key is the account id, value is the co-followed vector of the key
	for (NamedVector entry : vectors) {
		vec.set(entry);
		writer.append(new Text(entry.getName()), vec);
	}
	writer.close();
		
}
{% endcodeblock %}

#####3. 使用Mahout中的K-Means进行聚类

在介绍如何使用Mahout的K-Means之前，先复习一下基本的K-Means算法，如下所述：

	**基本的K-Means算法**

	1, 选择K个点作为初始质心

	2, repeat:

	3,	将每个点指派到最近的质心，形成K个簇

	4,	重新计算每个簇的质心

	5, until 质心不再发生改变


{% codeblock lang:java %}

//write initial clusters point, in this case 5 points/vectors
Path path = new Path("data/clusters/part-00000");
SequenceFile.Writer writer = new SequenceFile.Writer(fs, conf, path,
		Text.class, Kluster.class);

for (int i = 0; i < k; i++) {
	NamedVector nVec =  vectors.get(i);
			
	Kluster cluster = new Kluster(nVec, i, new CosineDistanceMeasure());
	writer.append(new Text(cluster.getIdentifier()), cluster);
}
writer.close();
		
//here runs the k-means clustering in mapreduce mode, set runSequential to false
KMeansDriver.run(conf, new Path("data/vectors"), new Path(
		"data/clusters"), new Path("output"),
		new CosineDistanceMeasure(), 0.001, 10, true, 0, false);
{% endcodeblock %}

上面的代码显示了Mahout中运行K-Means算法的用法，首先提供K个初始质心，如果用户没有提供初始质心，Mahout将根据用户给出的K随机的选择K个点作为初始质心。

KMeansDriver提供了运行K-Means算法的入口，其中的参数包含：

	/**
   	* Iterate over the input vectors to produce clusters and, if requested, use the results of the final iteration to
   	* cluster the input vectors.
   	* 
   	* @param input
   	*          the directory pathname for input points
   	* @param clustersIn
   	*          the directory pathname for initial & computed clusters
   	* @param output
   	*          the directory pathname for output points
   	* @param measure
   	*          the DistanceMeasure to use
   	* @param convergenceDelta
   	*          the convergence delta value
   	* @param maxIterations
   	*          the maximum number of iterations
   	* @param runClustering
   	*          true if points are to be clustered after iterations are completed
   	* @param clusterClassificationThreshold
   	*          Is a clustering strictness / outlier removal parameter. Its value should be between 0 and 1. Vectors
   	*          having pdf below this value will not be clustered.
   	* @param runSequential
   	*          if true execute sequential algorithm, else run algorithm in mapreduce mode 
   	*/


####总结

本文简要的介绍了Mahout以及聚类分析，并使用实例说明了如何使用Mahout的K-Means进行聚类分析。总的来说，使用Mahout进行聚类分析需要用户将数据转换成Mahout的Vector对象，并且写成Sequence文件格式，然后选择初始的质心和适当的距离度量调用KMeansDriver进行聚类。

当然聚类分析还有很多的内容，比如如何选择距离度量，K值如何确定以及如何度量聚类的好坏等等，本篇文章旨在介绍如何使用Mahout进行聚类分析，更多的关于聚类以及机器学习的知识请您阅读相关的专业书籍。


####参考数目

\[1\] Sean Owen etc., Mahout in Action, Manning Publications, 2011

\[2\] Pang-Ning Tan等, 数据挖掘导论, 人民邮电出版社, 2011 

\[3\] 文中代码实例的源码地址[CoFollowedClustering.java][src]

`---EOF---`


[src]:https://github.com/yoyzhou/weibo_analysis/blob/master/src/com/yoyzhou/weibo/CoFollowedClustering.java
[wk]:http://www.cs.waikato.ac.nz/ml/weka/
[rdm]:https://rapid-i.com/content/view/181/190/

