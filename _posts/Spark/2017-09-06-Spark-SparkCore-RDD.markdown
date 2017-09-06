---
Author: Hywel
layout: post
title: Spark RDD(带源码)
description: 
date: 星期三, 06. 九月 2017  8:31下午
categories: Spark
---

## Spark RDD
Spark RDD（Resilient Disttributed Dataset）是整个Spark的核心基础，上层的Spark Streaming，Spark SQL等都依赖于RDD。Spark SQL里的DataFrame在执行时，最终也会转换为一系列的RDD操作。所以理解RDD非常重要，里边包含Spark的容错机制，分区，计算等核心内容。

推荐先看下RDD的论文，[resilient distributed datasets a fault-tolerant abstraction for in-memory cluster computing](https://www.usenix.org/system/files/conference/nsdi12/nsdi12-final138.pdf)。由于之前关于这篇论文的笔记已经丢失，这次就结合源码做一些大概的总结。如有遗漏或者不正确的地方，欢迎大家指出。

## RDD包含内容
实际上RDD包含五个部分：
 * A list of partitions 一个分区集
 * A function for computing each split 每个分片的计算函数
 * A list of dependencies on other RDDs 从其他RDD转换而来的一系列路径依赖
 * Optionally, a Partitioner for key-value RDDs (e.g. to say that the RDD is hash-partitioned) 可选，key-value RDDs的分区器
 * Optionally, a list of preferred locations to compute each split on (e.g. block locations for an HDFS file) 可选，每个分片的位置信息（数据块位置等）

用户也可以自定义自己的RDD，不过自定义RDD必须实现这些属性和方法
### 1. 获取分区
```
/**
 * Implemented by subclasses to return how this RDD depends on parent RDDs. This method will only
 * be called once, so it is safe to implement a time-consuming computation in it.
 */
protected def getDependencies: Seq[Dependency[_]] = deps
```
array[partition]必须满足`rdd.partitions.zipWithIndex.forall { case (partition, index) => partition.index == index }`



### 2.给定分区的计算方法
```
/**
 * :: DeveloperApi ::
 * Implemented by subclasses to compute a given partition.
 */
@DeveloperApi
def compute(split: Partition, context: TaskContext): Iterator[T]
```

### 3.获取lineage依赖关系
```
/**
 * Implemented by subclasses to return how this RDD depends on parent RDDs. This method will only
 * be called once, so it is safe to implement a time-consuming computation in it.
 */
protected def getDependencies: Seq[Dependency[_]] = deps
```

### 4.（可选）重载获取位置元信息方法
```
/**
 * Optionally overridden by subclasses to specify placement preferences.
 */
protected def getPreferredLocations(split: Partition): Seq[String] = Nil
```

### 5.（可选）重载分区器
```
/** Optionally overridden by subclasses to specify how they are partitioned. */
@transient val partitioner: Option[Partitioner] = None
```

## RDD通用属性和方法
RDD包含一系列通用属性（sparkContext,RDD id,RDD name等）和操作方法

### 1. persist和cache方法
spark推荐多多使用cacha或者persist方法来缓存复用的数据集，达到提高运行效率的目的。所以persist和cache也是RDD应该有的一个通用方法

persist最基本的方法**persist(newLevel: StorageLevel, allowOverride: Boolean)**：
```
/**
 * Mark this RDD for persisting using the specified level.
 *
 * @param newLevel the target storage level
 * @param allowOverride whether to override any existing level with the new one
 * 第一个参数传入缓存级别，第二个参数控制是否支持更新缓存级别
 */
 private def persist(newLevel: StorageLevel, allowOverride: Boolean): this.type = {
		...
 }
```

其余的persist函数，最终都会调用上一个基本方法  
如**persist(newLevel: StorageLevel)**
```
 //实际调用persist(newLevel, allowOverride = false)
 def persist(newLevel: StorageLevel): this.type = {...}
```

如**persist()**，（persist函数不带参数默认设置缓存级别为MEMORY_ONLY）
```
 //实际调用persist(StorageLevel.MEMORY_ONLY)
 def persist(): this.type = persist(StorageLevel.MEMORY_ONLY)
```

而**cache()**函数，本质上也是persist函数，只是多了一次包装  
```
 def cache(): this.type = persist()
```

既然有缓存，那么肯定还有**unpersist**，移除缓存
```
/**
 * Mark the RDD as non-persistent, and remove all blocks for it from memory and disk.
 *
 * @param blocking Whether to block until all blocks are deleted.
 * @return This RDD.
 * 调用sc的unpersistRDD函数，并将该RDD的storageLevel标记为NONE
 */
 def unpersist(blocking: Boolean = true): this.type = {
   logInfo("Removing RDD " + id + " from persistence list")
   sc.unpersistRDD(id, blocking)
   storageLevel = StorageLevel.NONE
   this
 }
```
