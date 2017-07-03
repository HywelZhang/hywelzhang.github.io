---
Author: Hywel
layout: post
title: 《Calculus》第二章 Derivatives导数
description: 
date: 星期一, 03. 七月 2017  8:54上午
categories: Maths
---
## Derivatives导数 
### 1. 斜率Tangents
先来思考一个问题，求如图figure-1中函数的切线，已知点P(a,f(a)),只需要求出该切线的斜率就可以得到切线方程。  
假设有一个不断接近P点的Q(x,f(x))点,Q不断接近P，但是永远不等于P点，我们可以计算出PQ直线的斜率\\[m_{PQ} = \frac{f(x) - f(a)}{x - a}\\]
如图Figure-2,当Q点越来越接近P（无限接近），我们可以通过lim求得P点切线的斜率 \\[m = \lim\limits_{x \to a}\frac{f(x) - f(a)}{x - a}\\]
<div style="text-align:center;">
<div  align="left" style="display:inline-block;width:300px;margin-right:50px;"><img src="/assets/image/postImg/Maths/calculus/chapter2/figure1.png" style="width:100%;border-width:0;" height = "200" alt="figure1"/><h5 style="text-align:center;margin:0;line-height:30px;">Figure-1</h5></div>
<div  align="right" style="display:inline-block;width:300px;"><img src="/assets/image/postImg/Maths/calculus/chapter2/figure2.png" style="width:100%;border-width:0;" height = "200" alt="figure2" /><h5 style="text-align:center;margin:0;line-height:30px;">Figure-2</h5></div>
</div>
对上面的式子，将x-a定义为h来表示为变化量，则可以写成\\[m = \lim\limits_{h \to 0}\frac{f(a+h)-f(a)}{h} = f'(a)\\]

### 2. 速率问题
如图Figure-3，x为时间，y为路程，\\(\frac{\Delta y}{\Delta x} = \frac{f(x_{2})-f(x_{1})}{x_{2}}-x_{1}\\)可以得到\\([x_{1},x_{2}]\\)区间的平均速度。当\\(\Delta x\\)无限趋近于0，此时的lim值就是P点时候的瞬时速率\\[\lim\limits_{\Delta x \to 0}\frac{\Delta y}{\Delta x} = \lim\limits_{x_{2} \to x_{1}}\frac{f(x_{2})-f(x_{1})}{x_{2}-x_{1}}\\]
<div style="text-align:center;">
<div  align="center" style="display:inline-block;width:500px;"><img src="/assets/image/postImg/Maths/calculus/chapter2/figure3.png" style="width:100%;border-width:0;" height = "400" alt="figure3"/><h5 style="text-align:center;margin:0;line-height:30px;">Figure-3</h5></div>
</div>

### 3. 导数
将上面两个问题扩展一下，把P点扩展到函数上任一点，则有导数方程\\[f'(x) = \lim\limits_{h \to 0}\frac{f(x+h) - f(x)}{h}\\]
对应上面的实际问题，导数表示函数上某一点的切线斜率，也表示瞬时速率（个人理解，f'(x)就是表示函数的变化率）  

**导数相关推论**:
1. 如果f'(a)存在，则说函数f在a点可导。如果在开区间(a,b)(a,b是无穷时也有效；为什么是开区间？因为如果极限存在需要满足左极限等于右极限，如果取到端点，则缺少一边的极限，不能判断极限是否存在))，任意一点都可导，则函数f在区间(a,b)上可导。
2. 如果f在a点可导，那么函数f一定在a点连续(\\(\lim\limits_{x \to a}f(x) = f(a)\\)即能得出连续，简略证明：\\(\lim\limits_{x \to a}f(x) = \lim\limits_{x \to a}[f(a) + (f(x) - f(a))]\\);再通过导数公式推出\\(\lim\limits_{x \to a}[f(x)-f(a)] = f'(a)*0 = 0\\);最终得到\\(\lim\limits_{x \to a}f(x) = f(a)\\))。这个推论反过来不适用，连续不一定可导。   
(个人理解：导数是函数变化率的表示，导数存在，那么a点左边导数等于右边导数，也就是变化趋势相同，那么函数在a点左右肯定是连贯的。而函数在a点连续，是判断函数在a点的左极限是否等于右极限，也就是判断函数a点左右的极限值是否等于a点“该有”的值。例如y=|x|，在0点不可导，因为左右趋势一个是1，一个是-1。而这个函数在0点是连续的，因为y=|x|在0点的左极限，右极限的值等于在0点的函数值)
  
几个不可导的情况(1.角 ; 2.间断不连续 ; 3.\\(\lim\limits_{x \to a}|f'(x)| = \infty\\))：
<div style="text-align:center;">
<div  align="left" style="display:inline-block;width:300px;margin-right:50px;"><img src="/assets/image/postImg/Maths/calculus/chapter2/figure4.png" style="width:100%;border-width:0;    " height = "200" alt="figure4"/><h5 style="text-align:center;margin:0;line-height:30px;">Figure-1</h5></div>
<div  align="right" style="display:inline-block;width:100%;"><img src="/assets/image/postImg/Maths/calculus/chapter2/figure5.png" style="width:100%;border-width:0;" height = "250"     alt="figure5" /><h5 style="text-align:center;margin:0;line-height:30px;">Figure-2</h5></div>
</div>
