---
Author: Hywel
layout: post
title: HBase的三种运行模式Get Start
date: 星期四, 23. 二月 2017 04:00下午 
categories: HBase 
---

## 1.本地模式
standalone模式下所有的HBase线程，Master,RegionServers和Zookeeper都运行在单个JVM上，文件系统使用本地文件系统。

> note: HBase0.94版本前，HBase需要本地环回地址，需要在/etc/hosts下配置 

```
127.0.0.1  localhost
127.0.0.1  ubuntu.ubuntu-domain  ubuntu
```

### 1.1 安装HBase
简略步骤：  
1.下载HBase包，解压  
2.把JDK和HBase路径配置到环境变量  
3.配置`conf/hbase-site.xml`,HBase的主配置文件  

```
<!--设置HBase和Zookeeper写入数据的目录-->
<!--默认会写入/tmp下新建的目录，但是重启会被删除-->
<configuration>
  <property>
    	<name>hbase.rootdir</name>
    	<value>file:///home/testuser/hbase</value>
  </property>
  <property>
    	<name>hbase.zookeeper.property.dataDir</name>
    	<value>/home/testuser/zookeeper</value>
  </property>
</configuration>
```	

HBase的数据目录会自动创建。  

4.`bin/start-hbase.sh`启动HBase，可以使用`jps`查看HMaster进程是否成功启动。[http://localhost:16010]()可以查看HBase的web界面。

### 1.2 HBase基本操作
1. `hbase shell`进入到HBase控制台
2. `help` 展示HBase shell帮助命令
3. `create`命令创建一张新表，需要指定表名了列簇名  
	`create 'test','cf'`
4. 展示表信息   
	`list ‘'test'`
5. 向表中插入一条数据  
	`put 'test','row1','cf:a','value1'`
6. 扫描表中全部数据  
	`scan 'test'`
7. 查询单条数据  
	`get 'test','row1'`
8. 禁用表和启用表（删表或者改变表设置时，需要先禁用表）  
	禁用表：`disable 'test'`   启用表：`enable 'test'`
9. 删表  
	`drop 'test'`
10. 使用`exit`退出hbase shell

## 2. 伪分布式模式
伪分布式模式和本地模式一样，HBase仍然完全运行在本地。不同在于，伪分布式模式下，HBase的守护进程（HMaster，HRegionServer和Zookeeper）分别运行在分离的不同进程中。本地模式下，这些进程全部运行在一个jvm进程中。默认情况下，`hbase.rootdir`仍然和前边所讲一样，默认是/tmp/。伪分布式模式下，你配置时可以选择hdfs路径，也可以继续配置使用本地路径。

### 2.1 基本操作
1. 假设你刚刚完成前边的试验，HBase正在运行，停止HBase
2. 配置HBase，编辑`hbase-site.xml`文件。
```
	<!--Note：启用分布式模式-->
	<property>
  	<name>hbase.cluster.distributed</name>
  	<value>true</value>
	</property>
	<!--配置hbase数据存储目录，该目录会自动创建，不用手动创建-->
	<property>
  	<name>hbase.rootdir</name>
  	<value>hdfs://localhost:8020/hbase</value>
	</property>
```
3. `start-hbase.sh`启动HBase，如果配置正确，成功启动。可以使用`jps`看到HMaster和HRegionServer进程正在运行。
4. 在HDFS检查HBase目录是否生成
```
	$ ./bin/hadoop fs -ls /hbase
	Found 7 items
	drwxr-xr-x   - hbase users          0 2017-01-25 18:58 /hbase/.tmp
	drwxr-xr-x   - hbase users          0 2017-01-25 21:49 /hbase/WALs
	drwxr-xr-x   - hbase users          0 2017-01-25 18:48 /hbase/corrupt
	drwxr-xr-x   - hbase users          0 2017-01-25 18:58 /hbase/data
	-rw-r--r--   3 hbase users         42 2017-01-25 18:41 /hbase/hbase.id
	-rw-r--r--   3 hbase users          7 2017-01-25 18:41 /hbase/hbase.version
	drwxr-xr-x   - hbase users          0 2017-01-25 21:49 /hbase/oldWALs
```
5. 创建表和基本操作，如同前边本地模式一样
6. 启动和停止一个备用HMaster服务（伪分布式上开启没有多大实际意义，这里只是为了理解和学习。使得能在真正的分布式上得到应用）  
HMaster服务控制着整个HBase集群。你可以最多开启9个备用HMaster服务，加一个主服务HMaster，一个可以开启是个HMaster。开启一个备用HMaster，仅仅只需要调用命令`local-master-backup.sh`。备用HMaster使用的端口，你需要通过从当前master端口往后推多少的游标尺度来指定。每个HMaster使用三个端口（默认16010,16020和16030），指定的端口移动游标就是基于这三个端口号。假如游标推移单位2，那么这个HMaster将会使用16012,16022和16032。下方将展示一个示例，开启三个备用HMaster服务，并且使用端口分别为16012/16022/16032,16013/16023/16033,16015/16025/16035
```
/bin/local-master-backup.sh 2 3 5
```
停止一个备用HMaster，杀掉对应的进程ID（PID）。PID存储在一个文件中，文件名例如`/tmp/hbase-USER-X-master.pid`,这个文件只存储有该HMaster进程的PID，你可以直接使用`kill -9`去杀掉该进程。X为端口的游标移动数。下边例子将会停止一个端口位移为1的HMaster进程
```
cat /tmp/hbase-testuser-1-master.pid |xargs kill -9
```
7. 启动和停止一个额外新增的RegionServers  
	HRegionServer管理着存储在StoreFiles中的数据。一般来讲，集群上每个节点上会运行有一个HRegionServer。你可以使用`local-regionservers.sh`命令来运行多个RegionServers。运行模式和`local-master-backup.sh`很类似，每个RegionServer也都需要指定端口位移。RegionServer需要两个端口，默认端口为16020和16030。然而由于和HMaster默认端口冲突，自从HBase1.0.0版本以后，位移的基本端口已经改为16200和16300。你可以在一个服务上运行最多99个额外的RegionServer，下面的示例将会展示启动4个额外RegionServers，运行端口为16202/16302等（基本端口16200/16300+2）
```
$ .bin/local-regionservers.sh start 2 3 4 5
```
	手动停止RegionServer，可以使用`local-regionservers.sh `命令加上`stop`和端口位移的参数，示例
```
$ .bin/local-regionservers.sh stop 3
```
8. 停止HBase  
像本地模式下一样，可以使用`bin/stop-hbase.sh`来停用HBase服务

## 3. 分布式模式 
实际上，你需要试着部署一个完全分布式的集群来感受HBase。在一个分布式集群中，集群包含多个节点，每个节点又都会运行一个或多个HBase线程。这些包括主和备用的Master实例，多个Zookeeper节点和多个RegionServer节点。
> Note：搭建一个分布式的HBase，首先要确定每台机器之间能够互相通信，相互之间没有防火墙规则或者其他会阻止机器之间交流的东西。如果出现`no route to host`,请检查你的防火墙和网络。

### 3.1 配置分布式Hbase集群
1. 集群间配置SSH互信（不多说，无非就是ssh-keygen -t rsa生成密匙，然后ssh-copy-id分发）
2. 修改`conf/regionservers`文件，将需要运行RegionServer服务的机器名称或IP添加进去。
3. 在conf下新建一个*back-masters*的文件，将需要作为backup的HMaster机器名hostname添加进去
4. 配置Zookeeper，在*conf/hbase-site.xml*中将Zookeeper信息添加进去。更多Zookeeper的配置则请参考Zookeeper官网。
```
	<property>
  	<name>hbase.zookeeper.quorum</name>
  	<value>node-a.example.com,node-b.example.com,node-c.example.com</value>
	</property>
	<property>
  	<name>hbase.zookeeper.property.dataDir</name>
  	<value>/usr/local/zookeeper</value>
	</property>
```
5. 拷贝这些信息到每台机器上，每台机器上都需要有相同的配置

### 3.2 分布式HBase集群基本操作
1. 开启集群，`bin/start-hbase.sh`  
	Zookeeper最先启动，接下来是master，然后是RegionServers，最后是backup masters。
2. 验证集群是否正常运行，可以在每个集群上运行`jps`查看运行中的进程
```
	$ jps
	15930 HRegionServer
	16194 Jps
	15838 HQuorumPeer
	16010 HMaster
```
	**HQuorumPeer**进程是Zookeeper的实例，用来控制和启动HBase。如果你像上边这样，使用的内置Zookeeper，会在每个节点上都有一个HQuorumPeer进程，比较适合用来做测试。当然生产集群上一般都是使用的外置Zookeeper集群，这样的生成的Zookeeper进程名称是**QuorumPeer**。
3. Web UI  
在HBase0.98版本以后，Web UI的访问端口Master的60010换成了16010，RegionServer的60030换成了16030端口。如果设置正常，成功启动，接下来，你就可以用过[http://master:16010]()来访问HBase的Web管理界面了



