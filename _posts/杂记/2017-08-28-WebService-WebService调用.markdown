---
Author: Hywel
layout: post
title: WebService 
description: WebService调用;scala调用WebService
date: 星期一, 28. 八月 2017  7:40下午
categories: 杂记
---

## 一.什么是Web Service
WebService是一种跨编程语言和跨操作系统平台的远程调用技术。大白话就是，去调用一个远程服务，这就要求跨语言和平台。例如，调用银行的支付接口等。

## 二.Web Service架构
WebService平台由SOAP,WSDL和UDDI三大技术构成。

### 1.SOAP（Simple Object Access Protocol，即简单对象访问协议）
交换数据的一种协议规范。包含一个基于XML的可扩展消息信封格式，还需同时绑定一个网络传输协议。这个协议通常是HTTP或HTTPS，但也可能是SMTP或XMPP。一个简单的例子来说明SOAP使用过程，一个SOAP消息可以发送到一个具有Web Service功能的Web站点，例如，一个含有房价信息的数据库，消息的参数中标明这是一个查询消息，此站点将返回一个XML格式的信息，其中包含了查询结果（价格，位置，特点，或者其他信息）。由于数据是用一种标准化的可分析的结构来传递的，所以可以直接被第三方站点所利用。

**SOAP协议 = HTTP(或其他)协议 + XML数据格式**

### 2.WSDL（Web服务描述语言，Web Services Description Language）
WSDL描述Web服务的公共接口。这是一个基于XML的关于如何与Web服务通讯和使用的服务描述，包括可调用的函数、参数和返回值等。

WSDL文件保存在Web服务器上，通过一个url地址就可以访问到它。客户端要调用一个WebService服务之前，要知道该服务的WSDL文件的地址。WebService服务提供商可以通过两种方式来暴露它的WSDL文件地址：1.注册到UDDI服务器，以便被人查找；2.直接告诉给客户端调用者。

### 3.UDDI(统一描述、发现和集成Universal Description, Discovery, and Integration)
一个用来发布和搜索WEB服务的协议，应用程序可借由此协议在设计或运行时找到目标WEB服务。通俗来讲，就是发布自己的服务到互联网上。

它是一个基于XML的跨平台的描述规范，可以使世界范围内的企业在互联网上发布自己所提供的服务。
UDDI是OASIS发起的一个开放项目，它使企业在互联网上可以互相发现并且定义业务之间的交互。UDDI业务注册包括三个元件：
白页：有关企业的基本信息，如地址、联系方式以及已知的标识；
黄页：基于标准分类的目录；
绿页：与服务相关联的绑定信息，及指向这些服务所实现的技术规范的引用。
UDDI是核心的Web服务标准之一。它通过简单对象存取协议进行消息传输，用Web服务描述语言描述Web服务及其接口使用

**重点概括：使用XML作为消息格式，并以SOAP封装，由HTTP传输**

**上面三个标准由这些组织制订：W3C负责XML、SOAP及WSDL；OASIS负责UDDI。**

## 三.Web Service调用
### 1.通过创建动态client调用。
这种方式简单直接，在运行时动态生成代码去远程调用。不用提前生成client的代码。

```
//创建动态client
JaxWsDynamicClientFactory dcf = JaxWsDynamicClientFactory.newInstance();
org.apache.cxf.endpoint.Client client = dcf.createClient("http://xxx.xxx.xxx.xxx:xx/subscriberService?wsdl");
//QName，指定namespace和调用方法
QName name = new QName("http://xxx.xxx.xxx/", "callMethods");
//调用方法
Object[] objects = client.invoke(params);

```

由于应用场景稍微比较特殊，使用scala编写的spark程序调用一个远程web service服务。本地测试虽然通过，但是上生产集群环境挂掉，在创建client时报错如下（如果有大神知道原因，希望能邮件我 godbewithyou1314@gmail.com,非常感谢）：
```
 11:41:48 ERROR yarn.ApplicationMaster: User class threw exception: java.lang.NullPointerException
java.lang.NullPointerException
	at org.apache.cxf.wsdl11.WSDLServiceFactory.<init>(WSDLServiceFactory.java:74)
	at org.apache.cxf.endpoint.dynamic.DynamicClientFactory.createClient(DynamicClientFactory.java:296)
	at org.apache.cxf.endpoint.dynamic.DynamicClientFactory.createClient(DynamicClientFactory.java:241)
	at org.apache.cxf.endpoint.dynamic.DynamicClientFactory.createClient(DynamicClientFactory.java:234)
	at org.apache.cxf.endpoint.dynamic.DynamicClientFactory.createClient(DynamicClientFactory.java:189)
	at com.sfexpress.ddt.util.SubscribeFvp.SubsribeByListWaybill(SubscribeFvp.java:12)
	at com.sfexpress.ddt.dataReceiver.PldPriorityReceiver$.main(PldPriorityReceiver.scala:139)
	at com.sfexpress.ddt.dataReceiver.PldPriorityReceiver.main(PldPriorityReceiver.scala)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at org.apache.spark.deploy.yarn.ApplicationMaster$$anon$2.run(ApplicationMaster.scala:637)

```

### 2.通过wdsl2java生成本地client

先将要远程调用的方法代码在本地生成，直接调用
```
HelloService service = new HelloService();
Hello client = service.getHelloHttpPort();
 
String result = client.sayHi("Joe");

```

通过wsdl生成本地代码：
1. 通过命令行
`wsdl2java -client HelloWorld.wsdl`
具体参见：[wsdl2java](http://cxf.apache.org/docs/wsdl-to-java.html)

2. 通过maven
现在项目pom.xml中加入cxf代码生成插件,安装请参照[using-cxf-with-maven](http://cxf.apache.org/docs/using-cxf-with-maven.html)

```
<plugin>
	<groupId>org.apache.cxf</groupId>
    	<artifactId>cxf-codegen-plugin</artifactId>
        <version>2.2.3</version>
        <executions>
        	<execution>
            	<id>generate-sources</id>
                <phase>generate-sources</phase>
                <configuration>
                	<sourceRoot>${basedir}/代码生成地址</sourceRoot>
                    <wsdlOptions>
                    	<wsdlOption>
                        	<wsdl>http://xxx.xxx.xxx.xxx/你的wsdl地址?wsdl</wsdl>
                        </wsdlOption>
                     </wsdlOptions>
                 </configuration>
                 <goals>
                 	<goal>wsdl2java</goal>
                 </goals>
             </execution>
         </executions>
</plugin>
```

然后执行`mvn generate-sources`，将会生成本地代码


