---
layout: post
title: "HBase基础知识"
date: 星期一, 13. 二月 2017 05:38下午 
categories: HBase
---

## 简介

**HBase和HDFS：**
	HDFS作为大数据下的默认持久化存储层，个人理解，相当于单台PC的文件系统。而HDFS已经提供了持久化存储的作用，为什么还需要HBase等NOSQL数据库？因为HDFS提供顺序读写，会比较耗时。而HBase，cassandra等提供随记访问读写的能力，如同传统数据库在单台PC上的作用一样，提供结构化数据快速检索和管理的能力。
	
**HBase和Hive：**
	Hive 适合用来对一段时间内的数据进行分析查询，例如，用来计算趋势或者网站的日志。Hive 不应该用来进行实时的查询（Hive 的设计目的，也不是支持实时的查询）。因为它需要很长时间才可以返回结果；HBase 则非常适合用来进行大数据的实时查询，例如 Facebook 用 HBase 进行消息和实时的分析。对于 Hive 和 HBase 的部署来说，也有一些区别，Hive 一般只要有 Hadoop 便可以工作。而 HBase 则还需要 Zookeeper 的帮助（Zookeeper，是一个用来进行分布式协调的服务，这些服务包括配置服务，维护元信息和命名空间服务）。再而，HBase 本身只提供了 Java 的 API 接口，并不直接支持 SQL 的语句查询，而 Hive 则可以直接使用 HQL（一种类 SQL 语言）。如果想要在 HBase 上使用 SQL，则需要联合使用 Apache Phonenix，或者联合使用 Hive 和 HBase。但是和上面提到的一样，如果集成使用 Hive 查询 HBase 的数据，则无法绕过 MapReduce，那么实时性还是有一定的损失。Phoenix 加 HBase 的组合则不经过 MapReduce 的框架，因此当使用 Phoneix 加 HBase 的组成，实时性上会优于 Hive 加 HBase 的组合，我们后续也会示例性介绍如何使用两者。最后我们再提下 Hive 和 HBase 所使用的存储层，默认情况下 Hive 和 HBase 的存储层都是 HDFS。但是 HBase 在一些特殊的情况下也可以直接使用本机的文件系统。例如 Ambari 中的 AMS 服务直接在本地文件系统上运行 HBase。
	
## HBase存储格式

1.**数据在RMDB**

|  ID		|     姓名     |     性别     |     密码    |    时间戳|
|:----------:|:------------:|:------------:|:-----------:|:------------:|
|     1	|     张三     |     男        |    111      |20160719|
|     2	|     李四     |     男	   |    222      |20160720|

<br>
2.**数据在HBase中** 

|Row-Key	|CF:Column-Key|时间戳|Cell Value|
|:----:|---|----|---|
|1	|info:name  | 123456789 |张三|
|1	|info:sex	   | 123456789	 |男   |
|1	|securt:pwd| 123456789  |111|
|2	|info:name | 123456789  |李四|
|2	|info:sex	   | 123456789	|男    |
|2	|securt:pwd| 123456789  |222|

## HBase基本命令

安装好HBase后，使用`hbase shell`进入HBase shell命令行。

**常用命令：**

1. 查看HBase中所有的表
> list 

2. 查看表结构
> describe 'TableNmae'

3. 创建表
> create 'tableName',{NAME=>'ColumnFamilyName',VERSIONS=>1,BLOCKCACHE=>true,BLOOMFILTER=>'ROW',COMPRESSION=>'SNAPPY',TTL => ' 259200 '},{SPLITS => ['1','2','3','4','5','6','7','8','9','a','b','c','d','e','f']}

	**<font color="fuchsia">VERSION</font>**
	
	数据版本数，HBase数据模型允许一个cell的数据为带有不同时间戳的多版本数据集，VERSIONS参数指定了最多保存几个版本数据，默认为1。假如某个用户想保存两个历史版本数据，可以将VERSIONS参数设置为2，再使用如下Scan命令就可以获取到所有历史数据：

	> scan 'tableName',{VERSIONS => 2}
	  
	  **<font color="fuchsia">BLOOMFILTER</font>**
	  
	  布隆过滤器，优化HBase的随机读取性能，可选值NONE|ROW|ROWCOL，默认为NONE，该参数可以单独对某个列簇启用。启用过滤器，对于get操作以及部分scan操作可以剔除掉不会用到的存储文件，减少实际IO次数，提高随机读性能。Row类型适用于只根据Row进行查找，而RowCol类型适用于根据Row+Col联合查找，如下：
Row类型适用于：get ‘tableName’,'row1'
RowCol类型适用于：get ‘tableName’,’row1′,{COLUMN => ‘Column’}
对于有随机读的业务，建议开启Row类型的过滤器，使用空间换时间，提高随机读性能。
	  
	**<font color="fuchsia">COMPRESSION</font>**
	
	数据压缩方式，HBase支持多种形式的数据压缩，一方面减少数据存储空间，一方面降低数据网络传输量进而提升读取效率。目前HBase支持的压缩算法主要包括三种：GZip | LZO | Snappy，下面表格分别从压缩率，编解码速率三个方面对其进行对比：

	Snappy的压缩率最低，但是编解码速率最高，对CPU的消耗也最小，目前一般建议使用Snappy
	
	**<font color="fuchsia">TTL</font>**
	
	数据过期时间，单位为**秒**，默认为永久保存。对于很多业务来说，有时候并不需要永久保存某些数据，永久保存会导致数据量越来越大，消耗存储空间是其一，另一方面还会导致查询效率降低。如果设置了过期时间，HBase在Compact时会通过一定机制检查数据是否过期，过期数据会被删除。用户可以根据具体业务场景设置为一个月或者三个月。示例中TTL => ‘ 259200’设置数据过期时间为三天
	
	**<font color="fuchsia">IN_MEMORY</font>**
	
	数据是否常驻内存，默认为false。HBase为频繁访问的数据提供了一个缓存区域，缓存区域一般存储数据量小、访问频繁的数据，常见场景为元数据存储。默认情况，该缓存区域大小等于Jvm Heapsize * 0.2 * 0.25 ，假如Jvm Heapsize = 70G，存储区域的大小约等于3.2G。需要注意的是HBase Meta元数据信息存储在这块区域，如果业务数据设置为true而且太大会导致Meta数据被置换出去，导致整个集群性能降低，所以在设置该参数时需要格外小心。
	
	**<font color="fuchsia">BLOCKCACHE</font>**
	
	是否开启block cache缓存，默认开启。
	
	**<font color="fuchsia">SPLITS</font>**
	
	region预分配策略。通过region预分配，数据会被均衡到多台机器，这样可以一定程度上解决热点应用数据量剧增导致系统自动split引起的性能问题。
	
	当一个table刚被创建的时候，Hbase默认的分配一个region给table。也就是说这个时候，所有的读写请求都会访问到同一个regionServer的同一个region中，这个时候就达不到负载均衡的效果了，集群中的其他regionServer就可能会处于比较空闲的状态。解决这个问题可以用pre-splitting,在创建table的时候就配置好，生成多个region。

	在table初始化的时候如果不配置，Hbase不知道如何去split region的，因为Hbase不知道应该那个row key可以作为split的开始点。如果我们可以大概预测到row key的分布，我们可以使用pre-spliting来帮助我们提前split region。不过如果我们预测得不准确的话，还是可能导致某个region过热，被集中访问，不过还好我们还有auto-split。最好的办法就是首先预测split的切分点，做pre-splitting,然后后面让auto-split来处理后面的负载均衡。
	
	HBase数据是按照rowkey按升序排列，为避免热点数据产生，一般采用hash + partition的方式预分配region，比如示例中rowkey首先使用md5 hash，然后再按照首字母partition为16份，就可以预分配16个region。
	
	（建表参照[范欣欣的博客](http://hbasefly.com/2016/03/23/hbase_create_table/)，感谢）
	
4. 插值
> put 'tableName' 'RowName','CF:ColumnName','value',ts1(时间戳，一般省略)

5. 查询
> get 'tableName','RowName'
>get 'tableName','RowName','CF:ColumnName'
> get 'tableName', 'RowName', {COLUMN => ['c1', 'c2', 'c3']}
> get 'tableName', 'RowName', {COLUMN => 'c1', TIMERANGE => [ts1, ts2], VERSIONS => 4}

*HBase的shell操作，一个大概顺序就是操作关键词后跟表名，行名，列名这样的一个顺序，如果有其他条件再用花括号加上。*

6. 全表扫描
> scan 'table'
> scan 'table', {COLUMNS => ['c1', 'c2'], LIMIT => 10, STARTROW => 'hywel'}
> scan 'table', {COLUMNS => 'c1', TIMERANGE => [1303668804, 1303668904]}
> scan 'table', {FILTER => "(PrefixFilter ('row2') AND (QualifierFilter (>=, 'binary:xyz'))) AND (TimestampsFilter ( 123, 456))"}
> scan 'table', {FILTER => org.apache.hadoop.hbase.filter.ColumnPaginationFilter.new(1, 0)}	

7. 删除数据
> delete 'table','row'
> delete 'table','row','column'

	deleteall可以进行整行的范围删除
	
8. 修改表结构
+ 修改表名
```
disable 'table'
alter 'table',NAME='newTableName'
enable 'NewTableName'
```
+ 修改列簇
> alter 'table', NAME => 'cf', VERSIONS => 5 (属性=>新属性值)

+ 删除列簇
>alter 't1', NAME => 'f1', VERSIONS => 5


