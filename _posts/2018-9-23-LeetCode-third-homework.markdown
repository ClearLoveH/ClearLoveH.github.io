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

### 题目描述：

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

    - 此题考点在于不使用除法，乘法，mod运算符来实现除法运算，很明显的，题目在暗示我们去使用减法来实现除法。
    - 思路是很简单的，但是实现起来是有很多的坑需要我们去填。
    - 原题目中有一个词组非常绕口，就是 `truncate toward zero `———— `数轴上向零的方向取整趋零截尾`  ，理解了一下就是对于小数部分我们是需要进行处理的，是往靠近0的方向来舍去，即只留下整数部分，不进行四舍五入的处理。
    - 具体实现的话其实不难，我们将计算过程分为两步：
        - 我们先判断一下两个计算数的符号，同负异正，先判断出最终计算结果的正负号。
        - 然后对两个计算数取绝对值，将问题转为两个正数的除法，再通过减法来将商求出。
    - 而对于两个正数的减法来实现除法，我们记商quotient=0，余数为dibisor = n，通过`满n加一`的思路，被除数不断的减去除数，每减一次我们就将商加一，直到被除数<=0为止。

- Code for C++:
```c++
        class Solution {
        public:
            int divide(int dividend, int divisor) {
                int sign = 1;
                int quotient=0;
                if(divisor == 1)
                    return dividend;
                else if(divisor == -1){
                    if(dividend == INT_MIN)
                        return INT_MAX;
                    else return -dividend;
                }
                if(divisor < 0 ){
                    sign = -sign;       
                    divisor = -divisor;
                }
                if(dividend < 0){
                    if(dividend == INT_MIN){
                        dividend = INT_MAX;
                        quotient += 1;
                        dividend -= divisor - 1;
                    }
                    else dividend = -dividend;
                    sign = -sign;
                }

                while(dividend > 0){
                    dividend -= divisor;
                    if(dividend < 0)
                        break;
                    quotient += 1;
                }
                return quotient * sign ;
            }
        };
```        
>注：这题有一个大坑就是int的范围是我们需要考虑的。针对Input的被除数为`-2147483648` 的情况，我们需要特殊考虑，INT的最小值为`[-2^31 , 2^31 - 1]`,我们的算法将被除数取绝对值后，这种情况就会导致overflow溢出，所以此刻我们需要特殊考虑。
