---
Author: Hywel
layout: post
title: "Java Future"
description: "JAVA, FUTURE, 异步"
date: 星期三, 18. 十一月 2021  08:00下午
categories: "Java"
---

[TOC]

# Future



## 结构

```mermaid
classDiagram
  class Future {
    <<interface>>
    + cancel(mayInterruptIfRunning)
    + isCancelled()
    + isDone()
    + get()
    + get(timeout)
  }


```





​	

