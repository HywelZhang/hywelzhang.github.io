---
layout: post
title: "分布式锁之Redlock--一文看懂如何使用redis实现分布式锁"
date: 星期四, 08. 五月 2021 17:19下午 
categories: redis
---



[TOC]

# Redlock

## 一、简介

**Redlock** : 全称Redis Distribute Lock。网上利用redis实现分布式锁的文章和方式铺天盖地，各显神通。本文已redis官网文档的实现算法为主，结合java版本实现框架[Redisson](https://github.com/mrniko/redisson)进行说明

## 二、目标

实现目标：

1. 排他性：保证同一时刻只能由一个客户端持有该锁
2. 可释放性：发生死锁，或者持锁客户端异常crash后，能够保证锁能够释放恢复（一般通过超时实现）
3. 可用性：少部分redis节点的crash，不会影响锁的获取和释放



## 三、实现方案

通过setnx（原子操作：if not exist set key）设置一个key和key的过期时间。如果设置成功，表示持锁成功。如果不成功，已存在该key，说明已经被其他client抢占。参考文档:[redlock](https://redis.io/topics/distlock)

#### 1. 单实例redis实现

使用setnx抢占锁，并设置过期时间，防止持锁client crash后，导致锁无法释放

```
SET lock_key unique_value NX PX 30000
// NX： 没有时新建
// PX： 设置过期时间，锁超时释放
```

使用del释放锁。**释放时，一定要先对锁的value进行比对，比对成功再进行del**

```
if redis.call("get",KEYS[1]) == unique_value then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

>  **Note：**释放锁时，需要比对unique_value。避免产生这样的错误case：A client通过set持有锁，但是A client因为某种原因被阻塞，倒是锁超时释放了。这时，B client 发现无锁，通过set获取唯一锁成功。这时A client执行完毕，释放锁。如果不对分布式锁设置每个唯一的value，就会把B client的锁给释放。



#### 2. 主从模式（存在重复获取到锁的问题）

**why can't？**

```
There is an obvious race condition with this model:
1. Client A 从master获取锁成功.
2. master crash，此时还未将锁同步到slave节点.
3. slave晋升为master.
4. Client B 从新的master上获取锁成功。导致A、B client冲突，获取到了同一个锁
```



#### 3. 多master集群模式

我们前边已经讨论过单实例redis实现分布式锁的可用性。所以，在多个master时，我们也保证和单机版redis上使用一样的操作来实现分布式锁，同时提供高可用特性。现在假设我们有N个redis master，如何**获取锁**呢？

1. 获取锁时，记录当前时间
2. 用相同的key和value，顺序的在N个实例上进行setnx(获取锁)。同时会为每个实例的写入设置一个超时时间，超时时间一定要小于key的过期时间。
3. 每写入一个实例，就会用当前时间减步骤1记录的时间，计算写入耗时。当且仅当写入实例数超过总集群一半以上(N/2 + 1)，并且写入耗时小于key的过期时间，那么就认为该数据写入成功，获取锁成功。

4. 如果判定获取锁成功，设置真实过期时间为：设置的过期时间 - 第3步中计算出来的写入耗时
5. 如果写入失败（写入耗时大于过期时间，无法超过半数以上机器成功等），那么会在所有实例上进行释放锁(del)操作

**释放锁:**

1. 删除所有实例上的key

   

## 四、Java版本实现

Redlock有多个语言版本的实现，其中redisson为Java版本实现，此处仅以redisson作为代码使用举例。参考文档[redisson](https://github.com/redisson/redisson#quick-start)：

1. 引入类库

```
<dependency>
   <groupId>org.redisson</groupId>
   <artifactId>redisson</artifactId>
   <version>3.15.4</version>
</dependency>  
```

2. 代码使用示例

```
// 1. 创建config，连接redis
Config config = new Config();
config.useClusterServers()
       // use "rediss://" for SSL connection
      .addNodeAddress("redis://127.0.0.1:7181");

// or read config from file
config = Config.fromYAML(new File("config-file.yaml")); 

// 2. 创建Redisson实例

// Sync and Async API
RedissonClient redisson = Redisson.create(config);

// 3. Get Redis based implementation of java.util.concurrent.ConcurrentMap
// 使用场景1：利用redis版本实现的分布式ConcurrentMap
RMap<MyKey, MyValue> map = redisson.getMap("myMap");

// 4. Get Redis based implementation of java.util.concurrent.locks.Lock
// 使用场景2：利用redis实现的分布式锁
RLock lock = redisson.getLock("myLock");
lock.lock(2, TimeUnit.SECONDS);
// 解锁
lock.unlock();
```

