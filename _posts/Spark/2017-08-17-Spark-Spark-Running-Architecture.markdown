---
layout: post
title: "Spark运行时架构"
date: 星期三, 08. 二月 2017 09:39上午 
categories: Spark
---
Spark程序提交作业主要有三种运行模式，Standalong，Spark On Yarn Cluster，Spark On Yarn Client模式，先就生产环境使用最多的Spark On Yarn Cluster模式

## Spark On Yarn Cluster

![SparkOnYarnCluster](/assets/image/postImg/Spark/Spark-yarn-cluster.jpg)

1. 客户端提交一个应用程序，请求一个Application ID。
2. 如果请求成功，返回一个Application ID，RM（Resource Manager）会决定在某个Container里运行Application Master，并在RM上注册。
3. Application Master向RM请求资源，拿到分配的NM（Node Manager）。
4. NM上初始化Container。
5. Container会定时发送心跳给Application Master，确定状态是否健康。
6. NN初始化和启动Excutor。
7. Application Master将job拆分后的task分配到Excutor上运行。
8. Excutor将输出返回给Application Master。
9. 程序结束时，在AM结束前，还会将应用程序日志持久化到HDFS上再释放。

## Spark On Yarn Client
和上述模式很相似，不同之处在于，drive是在客户端，支持客户端与driver交互

## Spark On Standalong
资源调度交给了Spark自身的Master去做，流程和Yarn类似