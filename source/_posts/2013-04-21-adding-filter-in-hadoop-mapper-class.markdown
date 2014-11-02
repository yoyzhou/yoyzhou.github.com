---
layout: post
title: "Adding Filter in Hadoop Mapper Class"
date: 2013-04-21 15:08
comments: true
categories: [Hadoop, MapReduce, Bigdata, Analytic]
keywords: Hadoop, MapReduce, Bigdata Analytic, Weibo, Mapper, Reduce, Outputs, Aanlysis, Disk Space
description: There is my solutions to tackle the disk spaces shortage problem I described in the previous post. The core principle of the solution is to reduce the number of output records at Mapper stage; the method I used is **Filter**, adding a filter, which I will explain later, to decrease the output records of Mapper, which in turn significantly decrease the Mapper's Spill records, and fundamentally decrease the disk space usages. After applying the filter, with 30,661 records. some 200MB data set as inputs, the total Spill Records is 25,471,725,  and it only takes about 509MB disk spaces!
---

There is my solutions to tackle the disk spaces shortage problem I described in the [previous post][prepost]. The core principle of the solution is to reduce the number of output records at Mapper stage; the method I used is **Filter**, adding a filter, which I will explain later, to decrease the output records of Mapper, which in turn significantly decrease the Mapper's Spill records, and fundamentally decrease the disk space usages. After applying the filter, with 30,661 records. some 200MB data set as inputs, the total Spill Records is 25,471,725,  and it **only** takes about 509MB disk spaces!

####Followed Filter

And now I'm going to reveal what's kinda Filter it looks like, and how did I accomplish that filter.
The true face of the **FILTER** is called  **Followed Filter**, it filters users from computing co-followed combinations if their followed number does not satisfy a certain number, called **Followed Threshold**. 

Followed Filter is used to reduce the co-followed combinations at Mapper stage. Say we set the followed threshold to 100, meaning users who doesn't own 100 fans(be followed by 100 other users) will be ignored during co-followed combinations computing stage(to get the actual number of the threshold we need analyze statistics of user's followed number of our data set).

####Reason

Choosing followed filter is reasonable because how many user follows is a metric of user's popularity/famousness.

####HOW

In order to accomplish it, we need:

**First**, counting user's followed number among our data set, which needs a new MapReduce Job;

**Second**, choosing a followed threshold after analyze the statistics perspective of followed number data set got in first step;

**Third**, using DistrbutedCache of Hadoop to cache users who satisfy the filter to all Mappers;

**Forth**, adding followed filter to Mapper class, only users satisfy filter condition will be passed into co-followed combination computing phrase;

**Fifth**, adding co-followed filter/threshold in Reducer side if necessary.


####Outcomes
Here is the Hadoop Job Summary, after applying the followed filter with followed threshold of 1000, that means only users who are followed by 1000 users will have the opportunity to co-followed combinations, compared with the Job Summary in my previous post, most all metrics have significant improvements:

Counter				|Map			|Reduce			|Total
Bytes Written			|0			|1,798,185		|1,798,185
Bytes Read			|203,401,876		|0			|203,401,876
FILE_BYTES_READ			|405,219,906		|52,107,486		|457,327,392
**_HDFS_BYTES_READ_**			|**_203,402,751_**		|0			|203,402,751
**_FILE_BYTES_WRITTEN_**		|457,707,759		|52,161,704		|**_509,869,463_**
HDFS_BYTES_WRITTEN		|0			|1,798,185		|1,798,185
Reduce input groups		|0			|373,680		|373,680
Map output materialized bytes	|52,107,522		|0			|52,107,522
Combine output records		|22,202,756		|0			|22,202,756
**_Map input records_**		|**_30,661_**			|0			|30,661
Reduce shuffle bytes		|0			|52,107,522		|52,107,522
Physical memory (bytes) snapshot|2,646,589,440		|116,408,320		|2,762,997,760
**_Reduce output records_**		|0			|**_373,680_**		|373,680
**_Spilled Records_**			|**_22,866,351_**		|2,605,374		|25,471,725
Map output bytes		|2,115,139,050		|0			|2,115,139,050
Total committed heap usage (bytes)|2,813,853,696	|84,738,048		|2,898,591,744
CPU time spent (ms)		|5,766,680		|11,210			|5,777,890
Virtual memory (bytes) snapshot	|9,600,737,280		|1,375,002,624		|10,975,739,904
SPLIT_RAW_BYTES			|875			|0			|875
**_Map output records_**		|**_117,507,725_**		|0			|117,507,725
Combine input records		|137,105,107		|0			|137,105,107
Reduce input records		|0			|2,605,374		|2,605,374

####P.S.
Frankly Speaking, chances are **I am on the wrong way to Hadoop Programming**, since I'm palying `Pesudo Distribution Hadoop` with my personal computer, which has 4 CUPs and 4G RAM, in real Hadoop Cluster disk spaces might never be a trouble, and all the tuning work I have done may turn into meaningless efforts. Before the Followed Filter, I also did some Hadoop tuning like customed Writable class, RawComparator, block size and io.sort.mb, etc.
 
`---EOF---`

[prepost]:/blog/2013/04/20/my-first-luckily-and-sad-hadoop-results/
