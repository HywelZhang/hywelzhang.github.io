---
Author: Hywel
layout: post
title: "【Maven】构建多模块maven开发项目"
description: "用Maven构建一个用于多模块的Spark开发项目" 
date: 星期一, 27. 三月 2017  9:34上午
categories: 杂记
---
### <font color='green'>为什么需要构建一个多模块开发框架？</font>
项目为什么需要划分成模块：
1. 当项目越来越大，每个模块越来越可能会引用一些相同的jar包，但是版本不一致，很容易造成项目的版本冲突
2. 项目模块之间用到的一些util类，在其他项目也可能会用到。将util独立成一个模块，由专人维护，减少重复工作。还可以打成jar包共享给其他项目使用，提高代码复用性。
3. 当项目越来越大，build时间越来越长。拆分成模块后，可以各模块自行build，打包。

多模块开发，便于多人协作，每个模块对应一个pom.xml，整体项目一个pom.xml。可以将共用的jar依赖抽象到项目pom.xml，统一版本管理。模块单独需要的jar依赖则放在自己模块的pom.xml，不会影响到其他模块，降低耦合。

### <font color='green'>一个简单的Maven多模块架构:</font>
```
---- Spark-parent
|-- pom.xml (pom)
|
|-- Spark-util
| |-- pom.xml (jar)
|
|-- Spark-module1
| |-- pom.xml (jar)
|
|-- Spark-module2
| |-- pom.xml (jar)
|
|-- Spark-module3
|-- pom.xml (jar)
```
项目的pom.xml配置文件(Spark-parent下的pom.xml)

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.test.hans</groupId>
    <artifactId>app-parent</artifactId>
	<!--打包方式，下属模块打包方式修改为jar-->
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
	<!--包含模块-->
    <modules>
        <module>app-util</module>
        <module>app-dao</module>
        <module>app-service</module>
        <module>app-web</module>
    </modules>
</project>
```
### <font color='green'>jar依赖继承</font>
如何将各模块需要的jar依赖抽象到项目pom.xml中呢？  
**项目pom.xml**

```
<!--dependencyManagement元素下的依赖不会自动引入子模块，保证灵活性-->
<!--如果不加dependencyManagement元素，dependencies下依赖会自动继承到所有子模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.test.hans</groupId>
            <artifactId>org-remote-service</artifactId>
            <version>1.0.26</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```
**子模块pom.xml**  
这样继承项目pom.xml中的依赖，版本由项目pom.xml控制
```
<dependencies>
	<!--继承项目pom.xml的jar依赖，版本由项目pom.xml中指定-->
    <dependency>
        <groupId>com.test.hans</groupId>
        <artifactId>org-remote-service</artifactId>
    </dependency>
	<!--子模块需要单独引用的jar依赖-->
	<dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.28</version>
        </dependency>
</dependencies>
```
### <font color='Green'>plugins继承</font>
当然build中也可以这么做

**项目pom.xml**

```
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <version>2.5</version>
                <configuration>
                    <encoding>${encoding}</encoding>
                 </configuration>
            </plugin>         
        </plugins>
     </pluginManagement>
</build>
```

**子模块pom.xml**

```
<build>
     <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
            </plugin>         
        </plugins>
</build>
```
这样就实现了共用依赖的抽取，解决版本冲突问题，同时也给各子模块重复的灵活性。

### <font color='Red'>完整的pomx.xml实例</font>
部分依赖删减，不过能理解整个意思就行了。

**项目pom.xml**
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.sfexpress</groupId>
  <artifactId>ddt-spark</artifactId>
  <version>1.0-SNAPSHOT</version>
  <modules>
    <module>home-page</module>
    <module>data-receive</module>
    <module>comments-analysis</module>
    <module>order-life</module>
    <module>sale-promot</module>
  </modules>
  <packaging>pom</packaging>
  <inceptionYear>2017</inceptionYear>
  <properties>
    <maven.compiler.source>1.5</maven.compiler.source>
    <maven.compiler.target>1.5</maven.compiler.target>
    <encoding>UTF-8</encoding>
    <scala.version>2.10.5</scala.version>
    <spark.version>1.6.1</spark.version>
    <alluxio.version>1.3.0</alluxio.version>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <repositories>
    <repository>
      <id>scala-tools.org</id>
      <name>Scala-Tools Maven2 Repository</name>
      <url>http://scala-tools.org/repo-releases</url>
    </repository>
  </repositories>

  <pluginRepositories>
    <pluginRepository>
      <id>scala-tools.org</id>
      <name>Scala-Tools Maven2 Repository</name>
      <url>http://scala-tools.org/repo-releases</url>
    </pluginRepository>
  </pluginRepositories>

  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-streaming_2.10</artifactId>
        <version>${spark.version}</version>
      </dependency>
      <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-sql_2.10</artifactId>
        <version>${spark.version}</version>
      </dependency>
      <dependency>
        <groupId>org.alluxio</groupId>
        <artifactId>alluxio-core-client</artifactId>
        <version>${alluxio.version}</version>
      </dependency>

    </dependencies>
  </dependencyManagement>

  <build>
    <pluginManagement>
      <plugins>
        <plugin>
          <groupId>org.scala-tools</groupId>
          <artifactId>maven-scala-plugin</artifactId>
          <!--<version>${scala.maven.version}</version>-->
          <executions>
            <execution>
              <id>compile</id>
              <goals>
                <goal>compile</goal>
              </goals>
              <phase>process-resources</phase>
            </execution>
          </executions>
          <configuration>
            <scalaVersion>${scala.version}</scalaVersion>
          </configuration>
        </plugin>

        <plugin>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.1</version>
          <configuration>
            <source>1.6</source>
            <target>1.6</target>
            <encoding>${project.build.sourceEncoding}</encoding>
          </configuration>
        </plugin>

        <plugin>
          <artifactId>maven-assembly-plugin</artifactId>
          <version>2.6</version>
          <configuration>
            <descriptorRefs>
              <descriptorRef>jar-with-dependencies</descriptorRef>
            </descriptorRefs>
          </configuration>
          <executions>
            <execution>
              <id>make-assembly</id>
              <phase>package</phase>
              <goals>
                <goal>single</goal>
              </goals>
            </execution>
          </executions>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>

  <reporting>
    <plugins>
      <plugin>
        <groupId>org.scala-tools</groupId>
        <artifactId>maven-scala-plugin</artifactId>
        <configuration>
          <scalaVersion>${scala.version}</scalaVersion>
        </configuration>
      </plugin>
    </plugins>
  </reporting>
</project>
```

**其中一个子模块pom.xml**

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.sfexpress</groupId>
        <artifactId>ddt-spark</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <artifactId>data-receive</artifactId>
    <inceptionYear>2017</inceptionYear>
    <packaging>jar</packaging>

    <dependencies>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>0.8.2.1</version>
        </dependency>
        <dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.3</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.28</version>
        </dependency>
        <dependency>
            <groupId>io.spray</groupId>
            <artifactId>spray-json_2.10</artifactId>
            <version>1.3.2</version>
        </dependency>
        <dependency>
            <groupId>org.jsoup</groupId>
            <artifactId>jsoup</artifactId>
            <version>1.7.3</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-streaming-kafka_2.10</artifactId>
            <version>1.6.1</version>
        </dependency>

        <dependency>
            <groupId>org.alluxio</groupId>
            <artifactId>alluxio-core-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-streaming_2.10</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.10</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.scala-tools</groupId>
                <artifactId>maven-scala-plugin</artifactId>
            </plugin>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
            </plugin>
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>

```
