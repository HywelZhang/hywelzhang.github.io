---
Author: Hywel
layout: post
title: "Linux Shell中的if判断及参数" 
description: "Linux shell中如何使用if语句？if判断中各种参数是什么意思"
date: 星期一, 06. 三月 2017  3:07下午
categories: "Linux"
---
## Shell常用特殊变量
经常会在shell命令中，看到`$0, $#, $*, $@, $?, $$`这样的取值，这些代表什么呢？  

|变量|&nbsp;&nbsp;&nbsp;&nbsp;含义|
|:--:|:--|
|$0|&nbsp;&nbsp;&nbsp;&nbsp;当前脚本的文件名|
|$n|&nbsp;&nbsp;&nbsp;&nbsp;传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2|
|$#|&nbsp;&nbsp;&nbsp;&nbsp;传递给脚本或函数的参数个数|
|$\*|&nbsp;&nbsp;&nbsp;&nbsp;传递给脚本或函数的所有参数|
|$@ |&nbsp;&nbsp;&nbsp;&nbsp;传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $\* 稍有不同|
|$?|&nbsp;&nbsp;&nbsp;&nbsp;上个命令的退出状态，或函数的返回值。成功返回0，失败返回1|
|$$|&nbsp;&nbsp;&nbsp;&nbsp;当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID|

`$*` 和 `$@` 都是将参数一个一个返回  
`"$*"`将所有参数当做一个整体字符串返回 , `"$@"`将参数一个一个返回

## 常用判断参数
在shell命令文件中还经常会看到类似与`if [ -z "${SPARK_HOME}" ]; then`这样的判断语句？是不是也和我一样很疑惑` -z `是什么含义？  
下面是几个常见的参数，供查询使用： 
```
-a file exists. 
-b file exists and is a block special file. 
-c file exists and is a character special file. 
-d file exists and is a directory. 
-e file exists (just the same as -a). 
-f file exists and is a regular file. 
-g file exists and has its setgid(2) bit set. 
-G file exists and has the same group ID as this process. 
-k file exists and has its sticky bit set. 
-L file exists and is a symbolic link. 
-n string length is not zero. 
-o Named option is set on. 
-O file exists and is owned by the user ID of this process. 
-p file exists and is a first in, first out (FIFO) special file or named pipe. 
-r file exists and is readable by the current process. 
-s file exists and has a size greater than zero. 
-S file exists and is a socket. 
-t file descriptor number fildes is open and associated with a terminal device. 
-u file exists and has its setuid(2) bit set. 
-w file exists and is writable by the current process. 
-x file exists and is executable by the current process. 
-z string length is zero. 
```

## 判断命令
shell中除了有上边这样用来判断文件是否存在的参数，当然还有判断两个数是否相等这样更常规的命令  
例如，`if [ $# -gt 0 ]`这样判断传入参数个数是否为0

|命令|&nbsp;&nbsp;&nbsp;&nbsp;含义|
|:--:|:--|
|-eq|&nbsp;&nbsp;&nbsp;&nbsp;等于|
|-ne|&nbsp;&nbsp;&nbsp;&nbsp;不等于|
|-gt|&nbsp;&nbsp;&nbsp;&nbsp;大于|
|-lt|&nbsp;&nbsp;&nbsp;&nbsp;小于|
|ge|&nbsp;&nbsp;&nbsp;&nbsp;大于等于|
|le|&nbsp;&nbsp;&nbsp;&nbsp;小于等于|


