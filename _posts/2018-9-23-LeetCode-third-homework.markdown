---
layout:     post
title:      "LeetCode problems 3"
subtitle:   "Divide Two Integers"
date:       2018-9-23 18:47
author:     "Heng"
header-img: "img/比尔吉沃特.jpg"
catalog: true
tags:
    - LeetCode
---

>中秋快乐，身体安康，写写算法。

---

# Divide Two Integers

>Difficulty: Medium

### Description:

- Given two integers dividend and divisor, divide two integers without using multiplication, division and mod operator.

- Return the quotient after dividing dividend by divisor.

- The integer division should truncate toward zero.

### Note:

- Both dividend and divisor will be 32-bit signed integers.
The divisor will never be 0.

- Assume we are dealing with an environment which could only store integers within the 32-bit signed integer range: [−231,  231 − 1]. For the purpose of this problem, assume that your function returns 231 − 1 when the division result overflows.

--- 

题目描述：

- 给定两个整数的被除数和除数，在不使用乘法、除法和mod运算符的情况下，将两个整数相除。
- 返回计算的商。
- 整数除法应该截断为零。（数轴上向零的方向取整趋零截尾）

---

#### Example:

    Input: dividend = 10, divisor = 3
    Output: 3

    Input: dividend = 7, divisor = -3
    Output: -2

#### My answer:

- 解题思路：

