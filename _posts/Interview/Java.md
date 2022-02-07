---
Author: Hywel
layout: post
title: "Java
description: "面试，面试题，Java"
date: 星期三, 18. 十一月 2021  08:00下午
categories: "Interview"
---

[TOC]

# java

### 语法

#### 1. try-catch-finally

1. finally是否任意情况都会执行？

   ```
   不一定，以下特殊情况不会执行finally
   1. try语句块中，使用System.exit(0)
   2. 执行try时，主机突然断电等情况
   notes：此外不管是堆溢出，还是栈溢出，只要jvm不死，就会执行finally语句
   ```

2. 在try和finally中都存在return语句，会发生什么情况？

   ```
   finally中的return会覆盖try中的return结果
   ```

   *示例*

   ![image-20211117144609573](/Users/baidu/Workshop/hywelzhang.github.io/_posts/Interview/img/image-20211117144609573.png)