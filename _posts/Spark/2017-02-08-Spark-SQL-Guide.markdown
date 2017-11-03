---
layout: post
title: "【Spark SQL】Spark SQL程序指导【译】"
description: "Spark2.1 SQL程序指导，DataFrame，DataSets"
date: 星期三, 08. 二月 2017 09:39上午 
categories: Spark
---

## 概述
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Spark SQL是一个处理结构化数据的模块。不同于基本的Spark RDD API。Spark SQL提供了更多的数据结构和计算信息，Spark SQL会使用这些额外的信息去进行一些更多的优化。我们可以通过SQL或者Dataset API来与Spark SQL进行交互，但是无论是你使用哪种方式或者语言去写一个计算逻辑，下层都是使用同样的执行引擎去执行。不依赖于某一种方式，意味这开发者能够很容易使用自然语言的方式在不同API之间来回转换，实现最终计算。

### SQL
Spark SQL不仅可以用来进行SQL查询，也可以从Hive中读取数据，对Hive的操作请参照Hive Tables部分。SQL运行的返回结果是一个Dataset/Dataframe。可以通过命令行，或者JDBC/ODBC来与SQL接口进行交互。 

### Datasets and DataFrames
Dataset是一个分布式的数据集合，在Spark1.6以后才提供。Dataset既有RDD的长处（强类型，lambda表达式），又有被SQL执行引擎优化的好处。Dataset能够被JVM结构化，然后通过函数转换（map,flatMap,filter等）操作。现在Dataset API支持Scala和Java，Python暂时不支持。但是由于Python的动态属性，许多Dataset API的功能都能够通过Python实现。R同Python一样。

Dataframe是有命名列的Dataset。DataFrame的概念相当于传统关系型数据库中的表或者R/Python中的data frame，不同在于Dataframe底层会有更多的优化。DataFrame能够从很多源建立而来，例如：结构化数据文件，Hive表，外部数据库，或者RDDs。DataFrame的API支持Scala，Java，Python和R。在Scala和Java中，DataFrame相当于Dataset的行集。Scala API中，DataFrame其实就是Dataset[Row]的别名。在Java API中则是等同于Dataset<set>。

整篇文档中，我们将会经常将DataFrame看作Scala/Java Dataset的行集。

## Getting Started

### 程序接入点: SparkSession
Spark所有功能的入口点是SparkSession类，使用SparkSession.builder()就能够创建一个基本的SparkSession。

```
import org.apache.spark.sql.SparkSession

val spark = SparkSession
  .builder()
  .appName("Spark SQL basic example")
  .config("spark.some.config.option", "some-value")
  .getOrCreate()

// For implicit conversions like converting RDDs to DataFrames
import spark.implicits._
```
SparkSession是Spark2.0才有，用于支持Hive，包括使用HiveQL，访问Hive UDFs，读取Hive表数据等。使用这些功能，你甚至都不需要安装Hive。

### 创建DataFrames
使用SparkSession，程序能够从RDD，Hive表，或者Spark的其他数据源创建DataFrame。

示例，从JSON温江创建一个DataFrame：

```
val df = spark.read.json("examples/src/main/resources/people.json")

// Displays the content of the DataFrame to stdout
df.show()
// +----+-------+
// | age|   name|
// +----+-------+
// |null|Michael|
// |  30|   Andy|
// |  19| Justin|
// +----+-------+
```

### 无类型Dataset操作(aka DataFrame Operations)
DataFrames在Scala，Java，Python和R都提供了对结构化数据指定数据域操作。

如同上面讲的，在Spark2.0当中，DataFrame在Scala和Java API中仅仅只是Dataset的行集。这种操作被称为“无类型转换”，对比强类型Scala/Java Dataset的“类型转换”。

一个使用Dataset处理结构化数据的小例子：

```
// This import is needed to use the $-notation
import spark.implicits._
// Print the schema in a tree format
df.printSchema()
// root
// |-- age: long (nullable = true)
// |-- name: string (nullable = true)

// Select only the "name" column
df.select("name").show()
// +-------+
// |   name|
// +-------+
// |Michael|
// |   Andy|
// | Justin|
// +-------+

// Select everybody, but increment the age by 1
df.select($"name", $"age" + 1).show()
// +-------+---------+
// |   name|(age + 1)|
// +-------+---------+
// |Michael|     null|
// |   Andy|       31|
// | Justin|       20|
// +-------+---------+

// Select people older than 21
df.filter($"age" > 21).show()
// +---+----+
// |age|name|
// +---+----+
// | 30|Andy|
// +---+----+

// Count people by age
df.groupBy("age").count().show()
// +----+-----+
// | age|count|
// +----+-----+
// |  19|    1|
// |null|    1|
// |  30|    1|
// +----+-----+
```
除了上面这样一些简单的列推断和计算表达式，Datasets还有一系列丰富的功能函数，包括字符串操作，日期操作，常用数学包等等。完整API功能请参照[DataFrame API](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.functions$) 

### 直接运行SQL查询
SparkSession的sql函数，能够在程序中运行SQL查询并且返回一个DataFrame结果集。

```
// Register the DataFrame as a SQL temporary view
df.createOrReplaceTempView("people")

val sqlDF = spark.sql("SELECT * FROM people")
sqlDF.show()
// +----+-------+
// | age|   name|
// +----+-------+
// |null|Michael|
// |  30|   Andy|
// |  19| Justin|
// +----+-------+
```

### 全局临时视图
Spark SQL中的临时视图（Temporary views）生命周期是会话范围，临时视图会在创建他的session结束时消失。如果你想要创建一个全部session都能够共享的临时视图，并且在整个Spark应用程序运行期间都有效，你可以创建一个全局临时视图（global temporary view）。全局临时视图会在运行期间保存在*global_tmep*的系统数据库，我们如果要访问需要加上前缀，例如，SELECT * FROM global_temp.view1。

```
// Register the DataFrame as a global temporary view
df.createGlobalTempView("people")

// Global temporary view is tied to a system preserved database `global_temp`
spark.sql("SELECT * FROM global_temp.people").show()
// +----+-------+
// | age|   name|
// +----+-------+
// |null|Michael|
// |  30|   Andy|
// |  19| Justin|
// +----+-------+

// Global temporary view is cross-session
spark.newSession().sql("SELECT * FROM global_temp.people").show()
// +----+-------+
// | age|   name|
// +----+-------+
// |null|Michael|
// |  30|   Andy|
// |  19| Justin|
// +----+-------+
```

### 创建Datasets
Datasets比较类似于RDDs，然而，**<font color='red'>Datasets既不是使用Java序列化，也不是使用Kryo序列化,而是使用特殊的译码器encoder来序列化，进行网络间传输</font>**。encoder和标准序列化都是将对象转换为字节，但是encoder是动态编码，使用的格式允许Spark不用将字节码反序列化成对象就能执行很多操作，例如fiterinf，sorting和hashing等。

```
// Note: Case classes in Scala 2.10 can support only up to 22 fields. To work around this limit,
// you can use custom classes that implement the Product interface
case class Person(name: String, age: Long)

// Encoders are created for case classes
val caseClassDS = Seq(Person("Andy", 32)).toDS()
caseClassDS.show()
// +----+---+
// |name|age|
// +----+---+
// |Andy| 32|
// +----+---+

// Encoders for most common types are automatically provided by importing spark.implicits._
val primitiveDS = Seq(1, 2, 3).toDS()
primitiveDS.map(_ + 1).collect() // Returns: Array(2, 3, 4)

// DataFrames can be converted to a Dataset by providing a class. Mapping will be done by name
val path = "examples/src/main/resources/people.json"
val peopleDS = spark.read.json(path).as[Person]
peopleDS.show()
// +----+-------+
// | age|   name|
// +----+-------+
// |null|Michael|
// |  30|   Andy|
// |  19| Justin|
// +----+-------+
```

### 和RDDs互转
Spark SQL支持两种不同的方式去将RDDs转换为Datasets。第一种方法：反射，使用一个指定的object去推断RDD内部的结构信息。如果你已经知道RDD的结构信息，那么使用这种方法会很简洁，节约很多代码，并且运行的很好。第二种方式：通过一个程序接口创建一个结构化的Datasets，而后将该Datasetsapply到RDD上。这种方式会更繁杂，但是这种方式允许你在不知道列和类型时构造Datasets。

#### 方式1：使用反射推断结构
Scala的Spark SQL接口支持将包含case class的RDD自动转换为DataFrame。case class定义了表结构。case class的参数就是使用反射获取列名。case class中也能够包含复杂类型，Seqs或者Arrays等。RDD能够被隐式转化为Dataframe，注册为表后，还能够直接使用SQL语句进行操作。

```
import org.apache.spark.sql.catalyst.encoders.ExpressionEncoder
import org.apache.spark.sql.Encoder

// For implicit conversions from RDDs to DataFrames
import spark.implicits._

// Create an RDD of Person objects from a text file, convert it to a Dataframe
val peopleDF = spark.sparkContext
  .textFile("examples/src/main/resources/people.txt")
  .map(_.split(","))
  .map(attributes => Person(attributes(0), attributes(1).trim.toInt))
  .toDF()
// Register the DataFrame as a temporary view
peopleDF.createOrReplaceTempView("people")

// SQL statements can be run by using the sql methods provided by Spark
val teenagersDF = spark.sql("SELECT name, age FROM people WHERE age BETWEEN 13 AND 19")

// The columns of a row in the result can be accessed by field index
teenagersDF.map(teenager => "Name: " + teenager(0)).show()
// +------------+
// |       value|
// +------------+
// |Name: Justin|
// +------------+

// or by field name
teenagersDF.map(teenager => "Name: " + teenager.getAs[String]("name")).show()
// +------------+
// |       value|
// +------------+
// |Name: Justin|
// +------------+

// No pre-defined encoders for Dataset[Map[K,V]], define explicitly
implicit val mapEncoder = org.apache.spark.sql.Encoders.kryo[Map[String, Any]]
// Primitive types and case classes can be also defined as
// implicit val stringIntMapEncoder: Encoder[Map[String, Any]] = ExpressionEncoder()

// row.getValuesMap[T] retrieves multiple columns at once into a Map[String, T]
teenagersDF.map(teenager => teenager.getValuesMap[Any](List("name", "age"))).collect()
// Array(Map("name" -> "Justin", "age" -> 19))
```

#### 方式2：程序式的指定结构
当case class不能够提前定义时（例如，从一个字符串，或者一个文本文件中读取记录的结构时，不同用户产生的记录匹配出来的结构都不一样），DataFrame能够通过下面三个步骤在程序中创建：
1. 从原生RDD创建一个RDD of Rows;
2. 使用*StructType*创建一个结构；
3. 使用SparkSession提供的createDataframe方法来将RDD of Rows应用到结构上。

For Example：

```
import org.apache.spark.sql.types._

// Create an RDD
val peopleRDD = spark.sparkContext.textFile("examples/src/main/resources/people.txt")

// The schema is encoded in a string
val schemaString = "name age"

// Generate the schema based on the string of schema
val fields = schemaString.split(" ")
  .map(fieldName => StructField(fieldName, StringType, nullable = true))
val schema = StructType(fields)

// Convert records of the RDD (people) to Rows
val rowRDD = peopleRDD
  .map(_.split(","))
  .map(attributes => Row(attributes(0), attributes(1).trim))

// Apply the schema to the RDD
val peopleDF = spark.createDataFrame(rowRDD, schema)

// Creates a temporary view using the DataFrame
peopleDF.createOrReplaceTempView("people")

// SQL can be run over a temporary view created using DataFrames
val results = spark.sql("SELECT name FROM people")

// The results of SQL queries are DataFrames and support all the normal RDD operations
// The columns of a row in the result can be accessed by field index or by field name
results.map(attributes => "Name: " + attributes(0)).show()
// +-------------+
// |        value|
// +-------------+
// |Name: Michael|
// |   Name: Andy|
// | Name: Justin|
// +-------------+
```



















