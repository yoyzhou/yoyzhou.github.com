---
layout: post
title: "使用RawComparator加速Hadoop程序"
date: 2013-05-13 23:06
comments: true
categories: [hadoop, bigdata, writable, java]
keywords: hadoop, bigdata, writable, java, Rawcomparator, writablecomparable, custom, mapreduce, writablecomparator
description: 在本文中我们要介绍的方法从另外一个方面加快程序的速度，这就是使用RawComparator加速Hadoop程序。
---

在前面两篇文章[\[1\]][hw1][\[2\]][hw2]中我们介绍了Hadoop序列化的相关知识，包括Writable接口与Writable对象以及如何编写定制的Writable类，深入的分析了Writable类序列化之后占用的字节空间以及字节序列的构成。我们指出Hadoop序列化是Hadoop的核心部分之一，了解和分析Writable类的相关知识有助于我们理解Hadoop序列化的工作方式以及选择合适的Writable类作为MapReduce的键和值，以达到高效利用磁盘空间以及**快速读写对象**。因为在数据密集型计算中，在网络数据的传输是影响计算效率的一个重要因素，选择合适的Writable对象不但减小了磁盘空间，而且更重要的是其减小了需要在网络中传输的数据量，从而加快了程序的速度。

在本文中我们介绍另外一种方法加快程序的速度，这就是**使用RawComparator加速Hadoop程序**。我们知道作为键（Key）的Writable类必须实现WritableComparable接口，以实现对键进行排序的功能。Writable类进行比较时，Hadoop的默认方式是先将序列化后的对象字节流反序列化为对象，然后再进行比较（compareTo方法），比较过程需要一个反序列化的步骤。RawComparator的做法是**不进行反序列化，而是在字节流层面进行比较**，这样就省下了反序列化过程，从而加速程序的运行。Hadoop自身提供的IntWritable、LongWritabe等类已经实现了这种优化，使这些Writable类作为键进行比较时，直接使用序列化的字节数组进行比较大小，而不用进行反序列化。

####RawComparator的实现

在Hadoop中编写Writable的RawComparator一般不直接继承RawComparator类，而是继承RawComparator的子类**WritableComparator**，因为WritableComparator类为我们提供了一些有用的工具方法，比如从字节数组中读取int、long和vlong等值。下面是上两篇文章中我们定制的MyWritable类的RawComparator实现，定制的MyWritable由两个VLongWritable对组成，为了添加RawComparator功能，Writable类必须实现WritableComparable接口，这里不再展示实现了WritableComparable接口的MyWritableComparable类的全部内容，而只是MyWritableComparable类中Comparator的实现，完整的代码可以在[github][weibo_analysis]中找到。

{% codeblock lang:java %}
...//omitted for conciseness
/**
 * A RawComparator that compares serialized VlongWritable Pair
 * compare method decode long value from serialized byte array one by one
 *
 * @author yoyzhou
 *
 * */
public static class Comparator extends WritableComparator {

	public Comparator() {
		super(MyWritableComparable.class);
	}

	public int compare(byte[] b1, int s1, int l1, byte[] b2, int s2, int l2) {

		int cmp = 1;
		//determine how many bytes the first VLong takes
		int n1 = WritableUtils.decodeVIntSize(b1[s1]);
		int n2 = WritableUtils.decodeVIntSize(b2[s2]);

		try {
			//read value from VLongWritable byte array
			long l11 = readVLong(b1, s1);
			long l21 = readVLong(b2, s2);

			cmp = l11 > l21 ? 1 : (l11 == l21 ? 0 : -1);
			if (cmp != 0) {

				return cmp;

			} else {

				long l12 = readVLong(b1, s1 + n1);
				long l22 = readVLong(b2, s2 + n2);
				return cmp = l12 > l22 ? 1 : (l12 == l22 ? 0 : -1);
			} 
		} catch (IOException e) {
				throw new RuntimeException(e);
		}
	}
}

static { // register this comparator
	WritableComparator.define(MyWritableComparable.class, new Comparator());
}

...
{% endcodeblock %}

通过上面的代码我们可以看到要实现Writable的RawComparator我们只需要重载WritableComparator的`public int compare(byte[] b1, int s1, int l1, byte[] b2, int s2, int l2)`方法。在我们的例子中，通过从VLongWritable对序列化后字节数组中一个一个的读取VLongWritable的值，再进行比较。

当然编写完compare方法之后，不要忘了为Writable类注册编写的RawComparator类。

####总结

为Writable类编写RawComparator必须对Writable本身序列化之后的字节数组有清晰的了解，知道如何从字节数组中读取Writable对象的值，而这正是我们前两篇关于**Hadoop序列化和Writable接口**的文章所要阐述的内容。

通过以上的三篇文章，我们了解了Hadoop Writable接口，如何编写自己的Writable类，Writable类的字节序列长度与其构成，以及如何为Writable类编写RawComparator来为Hadoop提速。

####参考资料

Tom White, Hadoop: The Definitive Guide, 3rd Edition 

[Hadoop序列化与Writable接口(一)][hw1]

[Hadoop序列化与Writable接口(二)][hw2]

`--EOF--`



[hw1]:/blog/2013/05/09/hadoop-serialization-and-writable-object-1/
[hw2]:/blog/2013/05/10/hadoop-serialization-and-writable-object-2/
[weibo_analysis]:https://github.com/yoyzhou/weibo_analysis

