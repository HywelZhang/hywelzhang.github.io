---
Author: Hywel
layout: post
title: "Shell脚本中，while循环使用ssh注意事项"
description: "Shell脚本中，While循环下使用ssh踩坑记录"
date: 星期三, 08. 三月 2017  3:45下午
categories: 
---
最近在写一个脚本，读取一个IP文件，遍历ssh后执行一些操作。但是很奇怪，永远在连上第一个IP以后，循环就结束了，不会对下面的IP进行遍历。

### 问题代码：
```
# $1是一个主机列表
cat $1 | while read LINE
do
	echo "---------$LINE--------"
	ssh $LINE mv /tmp/test.txt /opt/
done
```

### 出现问题：
永远只会连接第一台机器，进行移动操作。列表中其余主机被略过。

### 问题原因：
while中使用重定向机制，$1 文件中的全部信息都已经读入并重定向给了while语句。所以当我们在while循环中再一次调用read语句，就会读取到下一条记录。问题就出在这里，ssh语句正好会读取输入中的所有东西，下面这个例子就能说明这个问题： 

```
cat $1|while read LINE
do
	echo "-------------$LINE---------"
	ssh 192.168.0.6 cat
done
```
上面代码运行结果：
![运行结果](/assets/image/Linux/2017-03-08-shellWhileResult.png) 
通过示例可以发现，ssh中的cat语句会打印出 $1 文件中的其他纪录，这就导致调用完ssh语句后，输入缓存中已经都被读完了，当read语句再读的时候当然也就读不到纪录，循环也就退出了。

### 如何解决？
**1. 将ssh的输入重定向** 
```
cat $1|while read LINE
do
	echo "-------------$LINE---------"
	ssh 192.168.0.6 ls < /dev/null
done
```

**2. 使用for循环实现**
```
for LINE in `cat $1`
do
	echo "-------------$LINE---------"
	ssh 192.168.0.6 cat
done

```

参考来源：[http://www.leeon.me/a/shell-while-ssh](http://www.leeon.me/a/shell-while-ssh) 
