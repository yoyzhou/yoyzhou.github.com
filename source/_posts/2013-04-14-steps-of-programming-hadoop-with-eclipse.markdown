---
layout: post
title: "使用Eclipse开发MapReduce程序的步骤"
date: 2013-04-14 20:58
comments: true
categories: [Hadoop, Bigdata, Programming, MapReduce, Eclipse]
keywords: Hadoop, Bigdata, Programming, MapReduce, Eclipse, Hadoop编程, MapReduce类, 大数据, Java编程
description: 使用Eclipse开发MapReduce程序时的路线，假定读者已经配置好了Hadoop环境并且了解Eclipse的相关操作。
---

以下8个步骤是我在使用Eclipse开发MapReduce程序时的路线，假定读者已经配置好了Hadoop环境并且了解Eclipse的相关操作。

步骤`0～4`为在Eclipse中编写和调试MapReduce程序；步骤`5、6`为在伪分布模式下运行MapReduce程序，并且通过导出项目到指定目录实现了Eclipse项目与Hadoop的关联。

0 创建Java项目

1 在项目的CLASS PATH中添加[Hadoop][hadoop]相关的JAR引用（注意在添加JAR文件，而不是JAR文件夹，要不然在4中会因为找不到JAR或者Class而报错）

> 如果你还下载了Hadoop的[源码][download]，也可以给Hadoop相关的JAR添加源码，这样在Eclipse就可以使用F3参看Hadoop源码）

2 按照[MapReduce][mapred]类规范，编写自己的MapReduce类

3 配置MapReduce类的运行参数

4 在Eclipse中以单机模式运行/调试程序

5 将程序导出（Export）为JAR文件到$HADOOP_HOME/lib下

6 在伪分布模式下运行程序 bin/hadoop jar lib/ur-exported-jar.JAR full-class-name 参数列表

> 例如，你导出的JAR文件名为myhadoop.jar，类名称com.coolcompany.wordcount，命令就是：bin/hadoop jar lib/myhadoop.jar com.coolcompany.wordcount 参数列表

7 部署程序到真实的Hadoop集群


`---EOF---`

[hadoop]: http://hadoop.apache.org/
[download]: http://hadoop.apache.org/releases.html
[mapred]: http://hadoop.apache.org/docs/r1.0.4/mapred_tutorial.html
