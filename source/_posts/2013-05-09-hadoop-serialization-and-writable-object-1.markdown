---
layout: post
title: "Hadoop序列化与Writable接口(一)"
date: 2013-05-09 20:49
comments: true
categories: [hadoop, bigdata, java, writable]
keywords: hadoop, bigdata, java, writable, mapreduce
description: 本文介绍Hadoop序列化以及如何编写定制的Hadoop Writable类，最后分析Hadoop中Writable实例占用的字节长度，以及Writable实例的字节序列的结构。
---

####序列化

**序列化**（serialization）是指将结构化的对象转化为字节流，以便在网络上传输或者写入到硬盘进行永久存储；相对的**反序列化**（deserialization）是指将字节流转回到结构化对象的过程。

在分布式系统中进程将对象序列化为字节流，通过网络传输到另一进程，另一进程接收到字节流，通过反序列化转回到结构化对象，以达到进程间通信。在Hadoop中，Mapper，Combiner，Reducer等阶段之间的通信都需要使用序列化与反序列化技术。举例来说，Mapper产生的中间结果（`<key: value1, value2...>`）需要写入到本地硬盘，这是序列化过程（将结构化对象转化为字节流，并写入硬盘），而Reducer阶段读取Mapper的中间结果的过程则是一个反序列化过程（读取硬盘上存储的字节流文件，并转回为结构化对象），需要注意的是，能够在网络上传输的只能是字节流，Mapper的中间结果在不同主机间洗牌时，对象将经历序列化和反序列化两个过程。

序列化是Hadoop核心的一部分，在Hadoop中，位于org.apache.hadoop.io包中的Writable接口是Hadoop序列化格式的实现。

####Writable接口

Hadoop Writable接口是基于DataInput和DataOutput实现的序列化协议，紧凑（高效使用存储空间），快速（读写数据、序列化与反序列化的开销小）。Hadoop中的键（key）和值（value）必须是实现了Writable接口的对象（键还必须实现WritableComparable，以便进行排序）。

以下是Hadoop（使用的是Hadoop 1.1.2）中Writable接口的声明：

{%codeblock lang:java%}
package org.apache.hadoop.io;

import java.io.DataOutput;
import java.io.DataInput;
import java.io.IOException;

public interface Writable {
  /** 
   * Serialize the fields of this object to <code>out</code>.
   * 
   * @param out <code>DataOuput</code> to serialize this object into.
   * @throws IOException
   */
  void write(DataOutput out) throws IOException;

  /** 
   * Deserialize the fields of this object from <code>in</code>.  
   * 
   * <p>For efficiency, implementations should attempt to re-use storage in the 
   * existing object where possible.</p>
   * 
   * @param in <code>DataInput</code> to deseriablize this object from.
   * @throws IOException
   */
  void readFields(DataInput in) throws IOException;
}

{%endcodeblock%}

####Writable类

Hadoop自身提供了多种具体的Writable类，包含了常见的Java基本类型（boolean、byte、short、int、float、long和double等）和集合类型（BytesWritable、ArrayWritable和MapWritable等）。这些类型都位于org.apache.hadoop.io包中。

![writable-classes](/images/hadoop-writable-class.png)

(图片来源：safaribooksonline.com)

####定制Writable类

虽然Hadoop内建了多种Writable类提供用户选择，Hadoop对Java基本类型的包装Writable类实现的RawComparable接口，使得这些对象不需要反序列化过程，便可以在字节流层面进行排序，从而大大缩短了比较的时间开销，但是当我们需要更加复杂的对象时，Hadoop的内建Writable类就不能满足我们的需求了(需要注意的是Hadoop提供的Writable集合类型并没有实现RawComparable接口，因此也不满足我们的需要)，这时我们就需要定制自己的Writable类，特别将其作为键（key）的时候更应该如此，以求达到更高效的存储和快速的比较。

下面的实例展示了如何定制一个Writable类，一个定制的Writable类首先必须实现Writable或者WritableComparable接口，然后为定制的Writable类编写write(DataOutput out)和readFields(DataInput in)方法，来控制定制的Writable类如何转化为字节流（write方法）和如何从字节流转回为Writable对象。

{% codeblock lang:java %}
package com.yoyzhou.weibo;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import org.apache.hadoop.io.VLongWritable;
import org.apache.hadoop.io.Writable;

/**
 *This MyWritable class demonstrates how to write a custom Writable class 
 *
 **/
public class MyWritable implements Writable{
		
		
	private VLongWritable field1;
	private VLongWritable field2;
		
	public MyWritable(){
		this.set(new VLongWritable(), new VLongWritable());
	}
		
		
	public MyWritable(VLongWritable fld1, VLongWritable fld2){
			
		this.set(fld1, fld2);
			
	}
		
	public void set(VLongWritable fld1, VLongWritable fld2){
		//make sure the smaller field is always put as field1
		if(fld1.get() <= fld2.get()){
			this.field1 = fld1;
			this.field2 = fld2;
		}else{
				
			this.field1 = fld2;
			this.field2 = fld1;
		}
		}
				
	//How to write and read MyWritable fields from DataOutput and DataInput stream
	@Override
	public void write(DataOutput out) throws IOException {
			
		field1.write(out);
		field2.write(out);
	}

	@Override
	public void readFields(DataInput in) throws IOException {
			
		field1.readFields(in);
		field2.readFields(in);
	}

	/** Returns true if <code>o</code> is a MyWritable with the same values. */
	@Override
	public boolean equals(Object o) {
		 if (!(o instanceof MyWritable))
		    return false;
		    
		    MyWritable other = (MyWritable)o;
		    return field1.equals(other.field1) && field2.equals(other.field2);
		    
	}
		
	@Override
	public int hashCode(){
			
		return field1.hashCode() * 163 + field2.hashCode();
	}
		
	@Override
	public String toString() {
		return field1.toString() + "\t" + field2.toString();
	}
		
}

{% endcodeblock %}


未完待续，下一篇中将介绍Writable对象序列化为字节流时占用的字节长度以及其字节序列的构成。


####参考资料

Tom White, Hadoop: The Definitive Guide, 3rd Edition 

`---To Be Continued---`








