---
Author: Hywel
layout: post
title: 【Spark Core源码解析】—— SparkConf 
description: "SparkConf类分析"
date: 星期二, 25. 四月 2017  3:05下午
categories: Spark
---
类： org.apache.spark.SparkConf

### 功能概述
1. 通过key-value键值对来设置Spark程序运行参数  
2. 大多数时候，使用`new SparkConf()`来新建，这个操作还会加载外部设置的spark.*的属性  
3. 通过SparkConf在程序里直接设置的属性，优先级会高于系统设置  
4. 在测试时，可以通过`new SparkConf(false)`来跳过加载外部环境属性  
5. 所有的setter方法都支持链式调用。例如，`new SparkConf().setMaster("local").setAppName("My app")`

### 代码解析

**以下代码仅为原始文件中部分源码，仅对方法级别进行分析**  

```
/**
 * @param loadDefaults 是否加载外部属性设置，默认为ture
 * @note 一旦将SparkConf对象传递给了spark，SparkConf会复制给各excutor，那么就不能再修改属性值。也就是说spark不支持运行时修改参数
 */
class SparkConf(loadDefaults: Boolean) extends Cloneable with Logging with Serializable {
	//保存键值对的HashMap
	private val settings = new ConcurrentHashMap[String, String]()	

	//创建SparkConf，并默认加载环境变量属性
 	def this() = this(true)
	
	//加载环境变量
	//参数silent：是否静默加载，默认fasle，会检测该key是否已废弃
	private[spark] def loadFromSystemProperties(silent: Boolean): SparkConf = {加载环境变量中以'spark.'开头的属性，再调用set(key, value, silent)添加到HashMap中}

	//设置键值对属性
	private[spark] def set(key: String, value: String, silent: Boolean): SparkConf = {判断key和value，如果silent==false，有null则打警告，然后settings.put(key, value)}
	def set(key: String, value: String): SparkConf = {set(key, value, false)}
	
	//设置master
	def setMaster(master: String): SparkConf = {set("spark.master", master)}

	//设置AppName
	def setAppName(name: String): SparkConf = { set("spark.app.name", name)}

	//设置需引用的Jar包，该jar包运行时会复制到集群上
	def setJars(jars: Seq[String]): SparkConf = {
      for (jar <- jars if (jar == null)) logWarning("null jar passed to SparkContext constructor")
      set("spark.jars", jars.filter(_ != null).mkString(","))
    }
	def setJars(jars: Array[String]): SparkConf = {setJars(jars.toSeq)}

	//设置executor环境变量
	def setExecutorEnv(variable: String, value: String): SparkConf = {set("spark.executorEnv." + variable, value)}
	def setExecutorEnv(variables: Seq[(String, String)]): SparkConf = {
      for ((k, v) <- variables) {
        setExecutorEnv(k, v)
      }
      this
    }
	def setExecutorEnv(variables: Array[(String, String)]): SparkConf = {setExecutorEnv(variables.toSeq)}

	//设置spark Home
	def setSparkHome(home: String): SparkConf = {set("spark.home", home)}
	
	//设置多个属性
	def setAll(settings: Traversable[(String, String)]): SparkConf = {
      settings.foreach { case (k, v) => set(k, v) }
      this
    }

	//设置时，如果属性不存在，检测是否该属性已废弃
	def setIfMissing(key: String, value: String): SparkConf = {
      if (settings.putIfAbsent(key, value) == null) {logDeprecationWarning(key)}
      this
    }

	//使用kryo序列化和register去处理指定class，如果调用多次，会将class追加
	def registerKryoClasses(classes: Array[Class[_]]): SparkConf = {...}

	//使用Kryo serialization和register来处理给定的Avro结构，这样能够序列化记录并显著减小磁盘IO
	def registerAvroSchemas(schemas: Schema*): SparkConf = {...}

	//移除某个属性
	def remove(key: String): SparkConf = {settings.remove(key);this}

	//获取属性值
	def get(key: String): String = {getOption(key).getOrElse(throw new NoSuchElementException(key))}

	//返回该Spark应用程序的application ID，在TaskScheduler注册及Executor启动以后，driver端可用
	def getAppId: String = get("spark.app.id")

	//是否包含指定参数
	def contains(key: String): Boolean = {settings.containsKey(key) || configsWithAlternatives.get(key).toSeq.flatten.exists { alt => contains(alt.key) }}

	//验证配置参数是否合法
	private[spark] def validateSettings() {...}
	
	...
}

private[spark] object SparkConf extends Logging {
	//已废弃的属性
	private val deprecatedConfigs: Map[String, DeprecatedConfig] = {...}
	//当前版本可用属性，如果已经废弃则弹出一个警告
	private val configsWithAlternatives = Map[String, Seq[AlternateConfig]](...)	
	private val allAlternatives: Map[String, (String, AlternateConfig)] = {
	  configsWithAlternatives.keys.flatMap { key =>
        configsWithAlternatives(key).map { cfg => (cfg.key -> (key -> cfg)) }
      }.toMap
	}
	
	//判断是否executor连接scheduler时所需配置属性，剩余属性将是driver端属性
	def isExecutorStartupConf(name: String): Boolean = {开头是"spark.auth"且通过安全验证||"spark.ssl"||"spark.rpc"||"spark.network"}
	
	//属性是否匹配`spark.*.port` 或者 `spark.port.*`
	def isSparkPortConf(name: String): Boolean = {...}
	
	//验证被废弃属性是否当前版本可用
	def getDeprecatedConfig(key: String, conf: SparkConf): Option[String] = {...}

	//如果属性已过时废除deprecated，打一个警告日志
	def logDeprecationWarning(key: String): Unit = {...}
	
	...
}
	
```

### 总结：
核心方法  
1. Set方法，同时会调用一系列验证方法去验证属性是否过时可用deprecated
2. get，获取属性值，包括加载进来的系统设置
