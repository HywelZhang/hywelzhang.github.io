---
Author: Hywel
layout: post
title: 《culculus》第一章 函数和极限
description: 
date: 星期四, 29. 六月 2017  2:38下午
categories: Maths 
---

>此系列博客旨在于重温微积分，所讲内容会在很浅的层面。一方面是想快速回忆一遍高数内容，另一方面在于博主也是菜鸡一枚。所以重在大纲和理解，不在于考研那样变态的抠字眼。此系列博客使用教材为《calculus》8th，本人英语也是渣，所以对于理解有偏差的地方也希望各位看官指出。在以后使用过程中，有新理解也会更新到博客中，请大家多多指教。

---

### 一. 基本术语

**对称性（奇偶函数）：** 如果函数f在x的定义域满足`f(-x)=f(x)`，则f被称为偶函数(even function)，图像关于y轴对称。如果函数f在x的定义域满足`f(-x)=-f(x)`，则f被称为奇函数(odd function)，图像关于原点对称。

**增函数和减函数：** 在区间[a,b]上，任取两点x1,x2且有x1<x2，如果恒有`f(x1)<f(x2)`，则说函数f在区间[a,b]上是增函数。如果恒有`f(x1)>f(x2)`，则函数f在区间[a,b]上是减函数。

### 二. 函数变换
函数的基本变换可用下面两个图归纳

函数平移：
![平移](/assets/image/postImg/Mathematics/calculus1/figure1-translate.png)

对于f(x)整体乘积，相当于对函数图像进行高矮处理：
![延伸](/assets/image/postImg/Mathematics/calculus1/figure2-stretch.png)

对于f(x)中x的乘积，相当于对图像的拉伸或者紧缩：
![对比](/assets/image/postImg/Mathematics/calculus1/figure3.png)

### 三. 复合函数
1. 对两个函数f(x),g(x)进行加减乘除运算。如(f+g)(x)=f(x)+g(x), (fg)(x)=f(x)g(x)。设f(x)和g(x)的定义域分别为A,B，则此时复合函数的定义域为 \\(A \cap B\\) 
2. 如果是y=f(u)=f(g(x))，其中一个函数为另一个函数的自变量。这种函数被称为*composition*。\\((f。g)(x)=f(g(x))\\)中，f(g(x))的定义域需要从g(x)的值域进行筛选后在结合g(x)的定义域确定。

### 四. 极限定义和性质
通俗来讲，就是假设f(x)中x无限趋近于一个数字a（x永远不等于a）时，我们有如下式子：\\[\lim\limits_{x \rightarrow +a}f(x) = L\\]
更通俗来讲，就是当\\(x \to a\\)时，有\\(f(x) \to L\\)，**Note** x永远不能等于a

**极限的精确定义**  
设函数f(x)在a的开区间上(不包含a点)有定义，如果\\(\forall \epsilon > 0\\)都有对应的\\(\delta > 0\\)使得  
\\(0 < |x-a| < \delta\\) 时有 \\(|f(x) - L| < \epsilon\\)  
则我们说\\[\lim\limits_{x \to a}f(x) = L \\]

### 五. 极限运算
假设 \\(\lim\limits_{x \to a}f(x)\\)和\\(\lim\limits_{x \to a}g(x)\\)存在，则有下列结算公式：  
1. \\(\lim\limits_{x \to a}[f(x)+g(x)]=\lim\limits_{x \to a}f(x) + \lim\limits_{x \to a}g(x)\\)  
2. \\(\lim\limits_{x \to a}[f(x)-g(x)]=\lim\limits_{x \to a}f(x) - \lim\limits_{x \to a}g(x)\\)  
3. \\(\lim\limits_{x \to a}[f(x)*g(x)]=\lim\limits_{x \to a}f(x) * \lim\limits_{x \to a}g(x)\\)  
4. \\(\lim\limits_{x \to a}\frac{f(x)}{g(x)}=\frac{\lim\limits_{x \to a}f(x)}{\lim\limits_{x \to a}g(x)}\\) (\\(\lim\limits_{x \to a}g(x)\not = 0\\))  
5. \\(\lim\limits_{x \to a}[cf(x)] = c\lim\limits_{x \to a}f(x)\\)  
6. \\(\lim\limits_{x \to a}[f(x)]^{n} = [\lim\limits_{x \to a}f(x)]^{n}\\) (n是正整数)  
7. \\(\lim\limits_{x \to a}c = c\\)  
8. \\(\lim\limits_{x \to a}x = a\\)   
9. \\(\lim\limits_{x \to a}x^{n} = a^{n}\\)(结合6和8可得；n是正整数)  
10. \\(\lim\limits_{x \to a}\sqrt[n]{x} = \lim\limits_{x \to a} \sqrt[n]{a}\\) (n是正整数)  
11. \\(\lim\limits_{x \to a}\sqrt[n]{f(x)} = \sqrt[n]{\lim\limits_{x \to a}f(x)}\\)(n是正整数；如果n是偶数，我们设\\(\lim\limits_{x \to a}f(x) > 0\\))  

几个相关推论：  
1. \\(\lim\limits_{x \to a}f(x) = L\\) 当且仅当 \\(\lim\limits_{x \to a^{-}}f(x) = L = \lim\limits_{x \to a^{+}}f(x)\\)  
2. 如果x靠近a（a为任意值）时，\\(f(x) \le g(x) \le h(x)\\) 且有 \\(\lim\limits_{x \to a}f(x) = \lim\limits_{x \to a}h(x) = L\\),则
\\[\lim\limits_{x \to a}f(x) = L\\]

记一个常见极限图像在这
![sinx/x](/assets/image/postImg/Mathematics/calculus1/figure4.png)


