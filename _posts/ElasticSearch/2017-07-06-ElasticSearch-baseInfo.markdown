---
Author: Hywel
layout: post
title: ElasticSearch基本操作
description: 
date: 星期四, 06. 七月 2017 10:20上午
categories: ElasticSearch
---
> 本文是学习《Elasticsearch 权威指南》而做的记录，有记录不清的地方，推荐直接去看原文http://learnes.net/getting_started/conclusion.html，在此感谢本书的翻译组

## 安装
```
## 1. 下载
curl -L -O http://download.elasticsearch.org/PATH/TO/LATEST/$VERSION.zip
unzip elasticsearch-$VERSION.zip
cd  elasticsearch-$VERSION

## 2. 运行
./bin/elasticsearch

## 3. 测试是否成功运行
curl 'http://localhost:9200/?pretty'

### if it success you should see this :
{
   "status": 200,
   "name": "Shrunken Bones",
   "version": {
      "number": "1.4.0",
      "lucene_version": "4.10"
   },
   "tagline": "You Know, for Search"
}
```
## 基本介绍
### 基本介绍
Elasticsearch 是一个分布式可扩展的实时搜索和分析引擎。它能帮助你搜索、分析和浏览数据，而往往大家并没有在某个项目一开始就预料到需要这些功能。Elasticsearch 之所以出现就是为了重新赋予硬盘中看似无用的原始数据新的活力。Elasticsearch 并不是单纯的全文搜索这么简单。还支持结构化搜索、统计、查询过滤、地理定位、自动完成等。  

Elasticsearch 是一个建立在全文搜索引擎 Apache Lucene(TM) 基础上的搜索引擎，Lucene 是当今最先进，最高效的全功能开源搜索引擎框架。但是Lucene非常复杂，所以诞生了ES对于Lucene的一个封装，方便开发人员使用。ES除了Lucene的搜索功能，还支持：  
+ 分布式实时文件存储，并将每一个字段都编入索引，使其可以被搜索。
+ 实时分析的分布式搜索引擎。
+ 可以扩展到上百台服务器，处理PB级别的结构化或非结构化数据。

### ES与传统数据库的对比
ES也分为几层目录，与传统数据库类比如下：
```
关系数据库     ⇒ 数据库 ⇒ 表    ⇒ 行    ⇒ 列(Columns)
Elasticsearch  ⇒ 索引   ⇒ 类型  ⇒ 文档  ⇒ 字段(Fields)
（这个索引是_index，名词意思，与后边的建索引意义不同）
```

例如建立一个员工信息表：
```
+ 为每一个员工的 文档 创建索引，每个 文档 都包含了一个员工的所有信息（下列1就是一个文档）。
+ 每个文档都会被标记为 employee 类型。 
+ 这种类型将存活在 megacorp 这个 索引 中。
+ 这个索引将会存储在 Elasticsearch 的集群中

curl -XPUT 'http://localhost:9200/megacorp/employee/1'
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```

## 基本操作
1. ES通过9300端口进行数据传输和通信，java可以直接通过节点客户端或传输客户端（9300端口）与ES进行交互，其他语言使用9200端口与ES的RESTful API进行通信

除了代码里通过上述客户端或者API与ES进行通信以外，当然还可以通过命令行命令curl操作  
### 基本命令格式
先来看一个简单的curl命令  
```
      <1>     <2>                   <3>    <4>
curl -XGET 'http://localhost:9200/_count?pretty' -d '
{  <5>
    "query": {
        "match_all": {}
    }
}
'
1. 相应的 HTTP 请求方法 或者 变量 : GET, POST, PUT, HEAD 或者 DELETE。
2. 集群中任意一个节点的访问协议、主机名以及端口。
3. 请求的路径。
4. 任意一个查询后再加上 ?pretty 就可以生成 更加美观 的JSON反馈，以增强可读性。
5. 一个 JSON 编码的请求主体（如果需要的话）。

```
上述命令返回结果：
```
{
    "count" : 0,
    "_shards" : {
        "total" : 5,
        "successful" : 5,
        "failed" : 0
    }
}
```
如果加上-i参数（`curl -i -XGET 'localhost:9200/'`）还能够显示head信息

**ES命令都是curl开头,大概格式一致，如`curl -XGET 'http://localhost:9200/_count?pretty'`，下文都简写为`GET /_count`**

### 检索文档
获取1这个文档的信息

```
GET /megacorp/employee/1
```

### 批量检索
 检索使用_search命令`GET /megacorp/employee/_search`，默认会返回最前的10条数据
### 指定条件检索
检索指定条件结果（给_search传递参数，就和网页搜索参数一样)match/_search?q=  
（检索姓为Smith的员工）

```
命令：
GET /megacorp/employee/_search?q=last_name:Smith

结果如下：
{
   ...
   "hits": {
      "total":      2,
      "max_score":  0.30685282,
      "hits": [
         {
            ...
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            ...
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```
上面还可以通过Query DSL(Domain Specific Language 领域特定语言)来进行搜索达到相同的效果  
```
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
```
### 结果过滤
在搜索中加入filer，对结果过滤  
检索姓Smith，并且年龄大于30岁的员工
```
GET /megacorp/employee/_search
{
    "query" : {
        "filtered" : {
            "filter" : {
                "range" : {
                    "age" : { "gt" : 30 } <1>
                }
            },
            "query" : {
                "match" : {
                    "last_name" : "Smith" <2>
                }
            }
        }
    }
}
```
### 全文匹配match  
检索about字段中有"rock climbing"的记录

```
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}
```
上述命令会把about字段中带有"rock climbing"或者带有"rock","climbing"的记录检索出来。当然会给结果做一个评分，都包含的结果会在更前面。

### 精确匹配match_phrase   
如果上面的实例，你不想要只包含"rock"这种部分匹配的结果，那么你可以使用match_phrase来进行精确匹配

```
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
这样就只会返回about字段中明确包含"rock climbing"字段的结果
```

### 高亮搜索结果highlight  
结果中会包含一个新的名叫highlight的部分，并将命中的部分高亮显示

```
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    },
    "highlight": {
        "fields" : {
            "about" : {}
        }
    }
}
```

### 统计汇总(类似group by)aggs  
统计员工兴趣爱好

```
GET /megacorp/employee/_search
{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}

运行结果：
{
   ...
   "hits": { ... },
   "aggregations": {
      "all_interests": {
         "buckets": [
            {
               "key":       "music",
               "doc_count": 2
            },
            {
               "key":       "forestry",
               "doc_count": 1
            },
            {
               "key":       "sports",
               "doc_count": 1
            }
         ]
      }
   }
}
```
当然还可以在汇总中加入其他操作，例如，只汇总姓Simth的员工（加入match）

```
GET /megacorp/employee/_search
{
  "query": {
    "match": {
      "last_name": "smith"
    }
  },
  "aggs": {
    "all_interests": {
      "terms": {
        "field": "interests"
      }
    }
  }
}
```
还可以进行多层面的汇总统计，随手统计一下每个兴趣的平均年龄

```
GET /megacorp/employee/_search
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}
结果如下：
 "all_interests": {
     "buckets": [
        {
           "key": "music",
           "doc_count": 2,
           "avg_age": {
              "value": 28.5
           }
        },
        {
           "key": "forestry",
           "doc_count": 1,
           "avg_age": {
              "value": 35
           }
        },
        {
           "key": "sports",
           "doc_count": 1,
           "avg_age": {
              "value": 25
           }
        }
     ]
  }
```
