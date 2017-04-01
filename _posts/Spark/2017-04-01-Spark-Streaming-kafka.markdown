---
Author: Hywel
layout: post
title: "Spark Streaming + Kafka "
description: "Spark Streaming读取kafka的两种方式，基于Receiver和基于kafka低阶api创建directDStreeam"
date: 星期六, 01. 四月 2017  3:06下午
categories: Spark
---
## <font color='red'>基于kafka低阶api的Direct访问方式(No Receivers)</font>
关于使用Direct Approach (No Receivers)方式来接收Kafka数据的好处我就不多讲了。长话短说：
1. 防止数据丢失。基于Receiver的方式，会启用一个接数线程将接收的数据暂时保存在excutor,另起线程处理数据。如果程序中途失败，在excutor中未来得及处理的数据将会丢失。所以基于Receiver的方式需要启用WAL机制来防止数据丢失。这样就会造成数据一次写两份，效率不够高效。
2. 与Receiver方式相比更加高效（原因如1中所讲）
3. kafka分区与接收过来的RDD分区一一对应，更符合逻辑，在不用重新分区时，能够提升效率。但是也有例外情况，当kafka分区比较少时，directDStream分区也相应比较少，这样并行度不够。repartition又会引发shuffle操作。所以需要自己权衡一下分区策略。

### 初步实现
_先申明本篇Blog使用的版本，注意适用范围（不说版本，上来就讲的都是耍流氓 --Spark1.6.1 --kafka-0.8)_  
Direct方式，会将每批读取数据的offset保存在流里边，所以如果不需要将offset写会基于zookeeper的监控工具中，实现起来超级简单  
首先，导包是必不可少的，我默认大家使用的是maven构建的项目，在pom.xml中添加依赖  

```
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming-kafka_2.10</artifactId>
    <version>1.6.1</version>
</dependency>
```
**示例：**

```
//需要导入kafka包
import org.apache.spark.streaming.kafka_
...

object directTest{
  def main(args: Array[String]) {
    val conf = new SparkConf().setAppName("kafka direct test")
    val sc = new SparkContext(conf)
    val ssc = new StreamingContext(sc,Seconds(10))
    /**
    *设置kafka的参数
    *metadata.broker.list 设置你的kafka brokers列表，一般是"IP:端口，IP:端口..."
    *
    *auto.offset.reset 这里没设置，默认为kafka读数从最新数据开始。
    *还有一个可选设置值smallest,会从kafka上最小的offset值开始读取
    */
    val kafkaParams = Map("metadata.broker.list" -> yourBrokers)
	val topic = "testTopic"
	directKafkaStream = KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder](ssc, kafkaParams, Set(topic))
	directKafkaStream.foreachRDD{
    }
  }
}
```
OK，到这里，你就已经把数给取到了，接下来用`directKafkaStream.foreachRDD`进行操作了，是不是超级简单？

但是问题来了，试想一下，生产环境上肯定都会有一个kafka监控工具，用direct的方式，你如果不把offset推回去，监控程序怎么能知道你数据消费没有？

### 进阶实现
你拿到了directDstream，官方文档只是简单的介绍了一下你可以通过`offsetRanges = rdd.asInstanceOf[HasOffsetRanges].offsetRanges`去获得该批数据的offset，但是没讲怎么推回去。虽然对大神来讲，so easy的问题。但是我这对zk(zookeeper简写，偷懒ing)编程又不熟的一小白开始搞的时候也是遇到很多问题。
下面我就用代码+注释的方式详细讲讲我是怎么实现的：  
当然，需要和zk协作，必须先加上zk的依赖

```
 <dependency>
    <groupId>com.101tec</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.3</version>
</dependency>
```

**代码及讲解：**

```
object directTest{
  def main(args: Array[String]) {
    val conf = new SparkConf().setAppName("kafka direct test")
    val sc = new SparkContext(conf)
    val ssc = new StreamingContext(sc,Seconds(10))
    
    //kafka基本参数,yourBrokers你的brokers集群
    val kafkaParams = Map("metadata.broker.list" -> yourBrokers)
    val topic = "testTopic"
    val customGroup = "testGroup"
	
    //新建一个zkClient，zk是你的zk集群，和broker一样，也是"IP:端口,IP端口..."
    /**
    *如果你使用val zkClient = new ZKClient(zk)新建zk客户端，
    *在后边读取分区信息的文件数据时可能会出现错误
    *org.I0Itec.zkclient.exception.ZkMarshallingError: 
    *  java.io.StreamCorruptedException: invalid stream header: 7B226A6D at org.I0Itec.zkclient.serialize.SerializableSerializer.deserialize(SerializableSerializer.java:37) at org.I0Itec.zkclient.ZkClient.derializable(ZkClient.java:740) ..
    *那么使用我的这个新建方法就可以了，指定读取数据时的序列化方式
    **/
    val zkClient = new ZkClient(zk, Integer.MAX_VALUE, 10000,ZKStringSerializer)
    //获取zk下该消费者的offset存储路径,一般该路径是/consumers/test_spark_streaming_group/offsets/topic_name	
    val topicDirs = new ZKGroupTopicDirs(fvpGroup, fvpTopic)
    val children = zkClient.countChildren(s"${topicDirs.consumerOffsetDir}")

    //设置第一批数据读取的起始位置
    var fromOffsets: Map[TopicAndPartition, Long] = Map()
    var directKafkaStream : InputDStream[(String,String)] = null

    //如果zk下有该消费者的offset信息，则从zk下保存的offset位置开始读取，否则从最新的数据开始读取（受auto.offset.reset设置影响，此处默认）
    if (children > 0) {
      //将zk下保存的该主题该消费者的每个分区的offset值添加到fromOffsets中
      for (i <- 0 until children) {
        val partitionOffset = zkClient.readData[String](s"${topicDirs.consumerOffsetDir}/$i")
        val tp = TopicAndPartition(fvpTopic, i)
        //将不同 partition 对应的 offset 增加到 fromOffsets 中
        fromOffsets += (tp -> partitionOffset.toLong)
        println("@@@@@@ topic[" + fvpTopic + "] partition[" + i + "] offset[" + partitionOffset + "] @@@@@@")
        val messageHandler = (mmd: MessageAndMetadata[String, String]) =>  (mmd.topic,mmd.message())
        directKafkaStream = KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder, (String,String)](ssc, kafkaParams, fromOffsets, messageHandler)
      }
    }else{
      directKafkaStream = KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder](ssc, kafkaParams, Set(fvpTopic))
    }

	/**
	*上边已经实现从zk上保存的值开始读取数据
	*下边就是数据处理后，再讲offset值写会到zk上
	*/
	//用于保存当前offset范围
    var offsetRanges = Array.empty[OffsetRange]
	val directKafkaStream1 = directKafkaStream.transform { rdd =>
	  //取出该批数据的offset值
      offsetRanges = rdd.asInstanceOf[HasOffsetRanges].offsetRanges
      rdd
    }.map(_._2).foreachRDD(rdd=>{
      //数据处理
	  ...
	
      //数据处理完毕后，将offset值更新到zk集群
      for (o <- offsetRanges) {
        val zkPath = s"${topicDirs.consumerOffsetDir}/${o.partition}"
        ZkUtils.updatePersistentPath(zkClient, zkPath, o.fromOffset.toString)
      }	
	})
  }
}
```

好的，基本操作已经完成，按照上边操作，已经能够实现direct方式读取kafka，并实现zk来控制offset。  
更多的细节优化，下次再更。。。

