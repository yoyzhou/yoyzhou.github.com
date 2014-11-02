---
layout: post
title: "Hadoop序列化与Writable接口(二)"
date: 2013-05-10 20:56
comments: true
categories: [hadoop, bigdata, java, writable]
keywords: hadoop, bigdata, java, writable, mapreduce, 大数据 
description: 本文通过实例介绍了Hadoop Writable类序列化时占用的字节长度，并分析了Writable类序列化后的字节序列的结构。需要注意的是Text类为了节省空间的目的采用了UTF-8的编码，而不是Java Character的UTF-16编码，自定义的Writable的字节序列与该Writable类的write()方法有关。最后指出，Writable是Hadoop序列化的核心，理解Hadoop Writable的字节长度和字节序列对于选择合适的Writable对象以及在字节层面操作Writable对象至关重要。
---

上一篇文章[Hadoop序列化与Writable接口（一）][hw1]介绍了Hadoop序列化，Hadoop Writable接口以及如何定制自己的Writable类，在本文中我们继续Hadoop Writable类的介绍，这一次我们关注的是Writable实例序列化之后占用的字节长度，以及Writable实例序列化之后的字节序列的构成。


####为什么要考虑Writable类的字节长度

大数据程序还需要考虑序列化对象占用磁盘空间的大小吗？也许你会认为**大数据**不是就是数据量很大吗，那磁盘空间一定是足够足够的大，一个序列化对象仅仅占用几个到几十个字节的空间，相对磁盘空间来说，当然是不需要考虑太多；如果你的磁盘空间不够大，还是不要玩大数据的好。

上面的观点没有什么问题，大数据应用自然需要足够的磁盘空间，但是能够尽量的考虑到不同Writable类占用磁盘空间的大小，高效的利用磁盘空间也未必就是没有必要的，选择适当的Writable类的另一个作用是**通过减少Writable实例的字节数，可加快数据的读取和减少网络的数据传输。**


####Writable类占用的字节长度

下面的表格显示的是Hadoop对Java基本类型包装后相应的Writable类占用的字节长度：

Java基本类型| Writable实现| 序列化后字节数 (bytes)
boolean| BooleanWritable |1
byte| ByteWritable |1
short| ShortWritable |2
int|IntWritable |4
|VIntWritable| 1–5
float| FloatWritable| 4
long |LongWritable| 8
|VLongWritable| 1–9
double|DoubleWritable| 8

不同的Writable类序列化后占用的字数长度是不一样的，需要综合考虑应用中数据特征选择合适的类型。对于整数类型有两种Writable类型可以选择，一种是定长（fixed-length）Writable类型,IntWritable和LongWritable；另一种是变长（variable-length）Writable类型，VIntWritable和VLongWritable。定长类型顾名思义使用固定长度的字节数表示，比如一个IntWritable类型使用4个长度的字节表示一个int；变长类型则根据数值的大小使用相应的字节长度表示，当数值在-112～127之间时使用1个字节表示，在-112～127范围之外的数值使用头一个字节表示该数值的正负符号以及字节长度（zero-compressed encoded integer）。

定长的Writable类型适合数值均匀分布的情形，而变长的Writable类型适合数值分布不均匀的情形，一般情况下变长的Writable类型更节省空间，因为大多数情况下数值是不均匀的，对于整数类型的Writable选择，我建议：

> 1\. 除非对数据的均匀分布很有把握，否则使用变长Writable类型

> 2\. 除非数据的取值区间确定在int范围之内，否则为了程序的可扩展性，请选择VLongWritable类型


####整型Writable的字节序列

下面将以实例的方式演示Hadoop整型Writable对象占用的字节长度以及Writable对象序列化之后字节序列的结构，特别是变长整型Writable实例，请看下面的代码和程序输出：

{% codeblock lang:java %}

package com.yoyzhou.example;

import java.io.*;
import org.apache.hadoop.io.*;
import org.apache.hadoop.util.StringUtils;

/**
 * Demos per how many bytes per each built-in Writable type takes and what does
 * their bytes sequences look like
 * 
 * @author yoyzhou
 * 
 */

public class WritableBytesLengthDemo {

	public static void main(String[] args) throws IOException {

		// one billion representations by different Writable object
		IntWritable int_b = new IntWritable(1000000000);
		LongWritable long_b = new LongWritable(1000000000);
		VIntWritable vint_b = new VIntWritable(1000000000);
		VLongWritable vlong_b = new VLongWritable(1000000000);

		// serialize writable object to byte array
		byte[] bs_int_b = serialize(int_b);
		byte[] bs_long_b = serialize(long_b);
		byte[] bs_vint_b = serialize(vint_b);
		byte[] bs_vlong_b = serialize(vlong_b);

		// print byte array in hex string and their length
		String hex = StringUtils.byteToHexString(bs_int_b);
		formatPrint("IntWritable", "1,000,000,000",hex, bs_int_b.length);

		hex = StringUtils.byteToHexString(bs_long_b);
		formatPrint("LongWritable", "1,000,000,000",hex, bs_long_b.length);

		hex = StringUtils.byteToHexString(bs_vint_b);
		formatPrint("VIntWritable", "1,000,000,000",hex, bs_vint_b.length);

		hex = StringUtils.byteToHexString(bs_vlong_b);
		formatPrint("VLongWritable", "1,000,000,000", hex, bs_vlong_b.length);
		
		
	}

	private static void formatPrint(String type, String param, String hex, int length) {

		String format = "%1$-50s %2$-16s with length: %3$2d%n";
		System.out.format(format, "Byte array per " + type
				+ "("+ param +") is:", hex, length);

	}

	/**
	 * Utility method to serialize Writable object, return byte array
	 * representing the Writable object
	 * 
	 * */
	public static byte[] serialize(Writable writable) throws IOException {

		ByteArrayOutputStream out = new ByteArrayOutputStream();
		DataOutputStream dataOut = new DataOutputStream(out);
		writable.write(dataOut);
		dataOut.close();

		return out.toByteArray();

	}

	/**
	 * Utility method to deserialize input byte array, return Writable object
	 * 
	 * */
	public static Writable deserialize(Writable writable, byte[] bytes)
			throws IOException {

		ByteArrayInputStream in = new ByteArrayInputStream(bytes);
		DataInputStream dataIn = new DataInputStream(in);
		writable.readFields(dataIn);

		dataIn.close();
		return writable;

	}
}

{% endcodeblock%}

程序输出：

	Byte array per IntWritable(1,000,000,000) is:  \     
	3b9aca00         with length:  4

	Byte array per LongWritable(1,000,000,000) is: \     
	000000003b9aca00 with length:  8

	Byte array per VIntWritable(1,000,000,000) is: \     
	8c3b9aca00       with length:  5

	Byte array per VLongWritable(1,000,000,000) is:\    
	8c3b9aca00       with length:  5



从上面的输出我们可以看出：

\+ 对1,000,000,000的表示不同的Writable占用了不同字节长度

\+ 变长Writable类型并不总是比定长类型更加节省空间，当IntWritable占用4个字节、LongWritable占用8个字节时，相应的变长Writable需要一个额外的字节来存放正负信息和字节长度。所以回到前面的整数类型选择的问题上，**选择出最合适的整数Writable类型，我们应该对数值的总体分布有一定的认识**。


####Text的字节序列

可以简单的认为Text类是java.lang.String的Writable类型，但是要注意的是Text类对于Unicode字符采用的是UTF-8编码，而不是使用Java Character类的UTF-16编码。

Java Character类采用遵循Unicode Standard version 4的UTF-16编码[\[1\]][char-encoding]，每个字符采用定长的16位(两个字节)进行编码，对于代码点高于Basic Multilingual Plane(BMP，代码点U+0000～U+FFFF)的增补字符，采用两个代理字符进行表示。

Text类采用的UTF-8编码，使用变长的1～4个字节对字符进行编码。对于ASCII字符只使用1个字节，而对于High ASCII和多字节字符使用2～4个字节表示，我想Hadoop在设计时选择使用UTF-8而不是String的UTF-16就是基于上面的原因，为了节省字节长度/空间的考虑。

> 由于Text采用的是UTF-8编码，所以Text类没有提供String那样多的操作，并且在操作Text对象时，比如Indexing和Iteration，一定要注意这个区别，不过我们建议在进行Text操作时，如果可能可以将Text对象先转换成String，再进行操作。

Text类的字节序列表示为一个VIntWritable + UTF-8字节流，VIntWritable为整个Text的字符长度，UTF-8字节数组为真正的Text字节流。具体请看下面的代码片段：

{% codeblock lang:java %}

...//omitted per conciseness
Text myText = new Text("my text");
byte[] text_bs = serialize(myText);
hex = StringUtils.byteToHexString(text_bs);
formatPrint("Text", "\"my text\"", hex, text_bs.length);
		
Text myText2 = new Text("我的文本");
byte[] text2_bs = serialize(myText2);
hex = StringUtils.byteToHexString(text2_bs);
formatPrint("Text", "\"我的文本\"", hex, text2_bs.length);
...

{% endcodeblock %}

程序输出：

	Byte array per Text("my text") is: \
 	076d792074657874 with length:  8

	Byte array per Text("我的文本") is: \
	0ce68891e79a84e69687e69cac with length: 13

在上面的输出中，首个字节代表的该段Text/文本的长度，在UTF-8编码下“my text”占用的字节长度为7个字节（07），而中文“我的文本”的字节长度是12个字节（0c）。


####定制Writable类的字节序列

本节中我们将使用上篇文章中的MyWritable类进行说明，回顾一下，MyWritable是一个由两个VLongWritable类构成的定制化Writable类型。

{% codeblock lang:java %}

...//omitted per conciseness
MyWritable customized = new MyWritable(new VLongWritable(1000),
 						new VLongWritable(1000000000));
byte[] customized_bs = serialize(customized);
hex = StringUtils.byteToHexString(customized_bs);
formatPrint("MyWritable", "1000, 1000000000", hex, customized_bs.length);
...

{% endcodeblock %}

程序输出：

	Byte array per MyWritable(1000, 1000000000) is: \
	8e03e88c3b9aca00 with length:  8

从输出我们可以很清楚的看到，定制的Writable类的字节序列实际上就是基本Writable类型的组合，输出“8e03e88c3b9aca00”的前三个字节是1000的VLongWritable的字节序列，“8c3b9aca00”是1000000000VLongWritable的字节序列，这一点可以从我们编写的MyWritable类的write方法中找到答案：

{% codeblock lang:java %}

...//omitted per conciseness
@Override
public void write(DataOutput out) throws IOException {
	field1.write(out);
	field2.write(out);
}
...

{% endcodeblock %}


####总结

本文通过实例介绍了Hadoop Writable类序列化时占用的字节长度，并分析了Writable类序列化后的字节序列的结构。需要注意的是Text类为了节省空间的目的采用了UTF-8的编码，而不是Java Character的UTF-16编码，自定义的Writable的字节序列与该Writable类的write()方法有关。

最后指出，Writable是Hadoop序列化的核心，理解Hadoop Writable的字节长度和字节序列对于选择合适的Writable对象以及在字节层面操作Writable对象至关重要。



####参考资料

Tom White, Hadoop: The Definitive Guide, 3rd Edition 

[Hadoop序列化与Writable接口(一)][hw1]


`---EOF---`









<!-- reference-like links -->
[hw1]:/blog/2013/05/09/hadoop-serialization-and-writable-object-1/
[char-encoding]:http://docs.oracle.com/javase/6/docs/api/java/lang/Character.html













