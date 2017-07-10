---
Author: Hywel
layout: post
title: 《Calculus》第三章 微分应用
description: 
date: 星期三, 05. 七月 2017 10:39上午
categories: Maths 
---
上一章已经讲了微分的相关知识，那么微分主要应用在哪些地方呢？本章就来讲讲应用的例子  
  
## 求最优解
微分一个最重要的应用就是解决**最优问题**，求解做某事的最优方式。下面将讨论几个问题

### 1. 求解最小花费？
### 2. 最大加速度？
### 3. 

## 极值，最值
1. 最大值(absolute maximum): \\(f(c) \ge f(x)\\)，x为定义域任一点
2. 最小值(absolute minimum): \\(f(c) \lt f(x)\\)，x为定义域任一点
3. 极大值(local maximum): \\(f(c) \ge f(x)\\)，x is near c
4. 极小值(local minimum): \\(f(x) \lt f(x)\\),x is near c

## Rolle Theorem（罗马定理）

> 假设f函数满足以下三个条件:
> 1. f在闭区间[a,b]上连续
> 2. f在开区间(a,b)上可导
> 3. f(a) = f(b)
> 那么在(a,b)之间一定有一点使得f'(c) = 0

证明：f'(c)=0,说明有极值，连续又可导的函数，如果f(a) = f(b),那么中间一点会有拐点，或者就是一条直线。所以很容易得出罗马定理。

## The Mean Value Theorem(中值定理)
> 假设f函数满足下列条件
> 1. f在闭区间[a,b]上连续
> 2. f在开区间(a,b)上可导
>  那么在（a,b）上一定有一点c满足 
> \\(f'(c) = \frac{f(b) - f(a)}{b - a}\\) 或者写成 \\(f(b) - f(a) = f'(c)(b - a)\\)

**推理:** 因为\\(m_{AB} = \frac{f(b) - f(a)}{b - a}\\)很明显是A,B两点间的斜率。所以这个定理讲得是，在（a,b）间至少有一点c的斜率与A,B的斜率相等，也就是说c点的切线和A,B平行。从下图，我们可以很方便的看出：  
!(figure1)[/assets/image/postImg/Maths/calculus/chapter3/figure1]

### little Theorem And Corollary: 
1. 如果f'(x) = 0在（a,b）上都成立，那么f一定是一个常数
2. 如果在区间（a,b）上对于所有x都有f'(x) = g'(x)，那么 f - g 一定在(a,b)上是一个常数。f(x) = g(x) + c,c是一个常数

## 一阶导数f'与函数f图像的联系
都知道函数f'(x)是函数f在点(x,f(x)) 处的斜率，那么f'(x)能给我们带来一些什么信息呢？  
### 1. f'(x)能够看出函数是增还是减
如下图所示，我们能够通过斜率也就是f'来判别函数f的增减
!(figure2)[/assets/image/postImg/Maths/calculus/chapter3/figure2] 
> a. 如果在某个区间内f'(x) > 0，那么函数f在该区间内单调递增
> b. 如果在某个区间内f'(x) < 0,那么函数f在该区间内单调递减

### 2. 判断极值问题
> a. 如果f'在c点由正变负,那么函数f在c点有极大值
> b. 如果f'在c点由负变正，那么函数f在c点有极小值
> c. 如果f'在c点的左右都是正数或者负数，那么函数f在c点没有极值

!(figure3)[/assets/image/postImg/Maths/calculus/chapter3/figure3] 

## 二阶导数f''与函数f图像的联系

