```
Author: Hywel
layout: post
title: "Java
description: "面试，面试题，算法"
date: 星期三, 18. 十一月 2021  08:00下午
categories: "Interview"
```

[TOC]

# 算法



## 位运算

### 1. SingleNumber （孤单数）

**题目描述：**给定一个int数组，其中的每个数字正好会出现两次，现在可能混入了一个只出现一次的数字。找出只出现一次的那个数字。

**示例：**nums = [1,2,1,3,2,5,3]    return:  5

**解题思路：异或**

```
使用位运算：异或
1. 异或交换率：a ^ b ^ c  = a ^ c ^ b
2. 异或等式： a ^ a = 0  , a ^ 0 = a
通过异或运算，相同数字异或后等于0，0异或任意数等于任意数的特性查找
(例如: a ^ b ^ c ^ a ^ b = c)

伪代码：
 int singleNum = 0;
 for (int num : nums) {
   singleNum ^= num;
 }
 return singleNum;
```



## Other

### 1. Integer to Chinese words （数字的中文读法）

**题目描述：**给一个int数字，输出中文读法。

**示例：** 输入： 1001   输出： 一千零一 

**解题思路：**

```
1. num % 10。 每次处理最后一位，转换为中文
2. 使用turn记录当前遍历次数，通过turn来选择计量单位。万、十万或者十亿等
3. 注意特殊情况，中间连续0，只用添加一个零，末尾连续0，需要跳过。负数先转正等

伪代码：
trun = 0;
while (num != 0) {
  int curNum = num % 10;
  if (curNum == 0) {0情况处理;}
	else { result += 计量单位(turn) + 中文数字(curNum);} // 从末尾处理，最后需要反转。所以先加单位，后加数字
	turn++;
	num = num / 10;
}
return result.reverse().addSign;
```



**实现代码：**

```
public static String speakInt(Integer num) {
    // 1. 边界处理： 0 和负数
    if (num == 0) {
        return "零";
    }
    String sign = num < 0 ? "负" : "";
    num = Math.abs(num);

    // 2. 定义数字和单位
    String[] numStrs = new String[]{"零", "一", "二", "三", "四", "五", "六", "七", "八", "九"};
    String[] baseStrs = new String[]{"", "十", "百", "千"};
    String[] highStrs = new String[]{"", "万", "亿"};

    StringBuilder result = new StringBuilder();
    // 位数，控制添加当前读取到哪位。千、百、十
    int turn = 0;
    while (num % 10 == 0) {
        turn++;
        num /= 10;
    }
    // 处理连续0情况
    int continueZeroCount = 0;
    while(num != 0) {
        int cur = num % 10;
        if (cur == 0) {
            continueZeroCount++;
            if (continueZeroCount == 1) { result.append("零"); }
        } else {
            continueZeroCount = 0;
            result.append(highStrs[turn / baseStrs.length]);
            result.append(baseStrs[turn % baseStrs.length]);
            result.append(numStrs[num % 10]);
        }
        num /= 10;
        turn++;
    }
    result.append(sign);
    return result.reverse().toString();
}
```

