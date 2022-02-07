---
Author: Robert
layout: post
title: "一致性算法——Gossip"
description: "分布式，一致性算法"
date: 星期三, 08. 十二月 2021  17:30下午
categories: "Distribute"
---

[TOC]

# Gossip

 Gossip protocol 也叫 Epidemic Protocol （流行病协议），是基于流行病传播方式的节点或者进程之间信息交换的协议，是一种去中心化的分布式协议。用于保证最终一致，但是“最终”的时间是一个无法明确的时间点。所以适合于AP场景的数据一致性处理，常见应用有：P2P网络通信、Apache Cassandra、[Redis](https://cloud.tencent.com/product/crs?from=10680) Cluster、Consul。

