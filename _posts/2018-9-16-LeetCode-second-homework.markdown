---
layout:     post
title:      "LeetCode problems 2"
subtitle:   "String to Integer (atoi)"
date:       2018-9-16 13:42
author:     "Heng"
header-img: "img/艾欧尼亚.jpg"
catalog: true
tags:
    - LeetCode
---

> 窗外台风很大，我们来聊聊算法。

---

#  String to Integer (atoi)

>Difficulty: Medium

### Description:

- Implement atoi which converts a string to an integer.

- The function first discards as many whitespace characters as necessary until the first non-whitespace character is found. Then, starting from this character, takes an optional initial plus or minus sign followed by as many numerical digits as possible, and interprets them as a numerical value.

- The string can contain additional characters after those that form the integral number, which are ignored and have no effect on the behavior of this function.

- If the first sequence of non-whitespace characters in str is not a valid integral number, or if no such sequence exists because either str is empty or it contains only whitespace characters, no conversion is performed.

- If no valid conversion could be performed, a zero value is returned.


#### Note:

- Only the space character ' ' is considered as whitespace character.
- Assume we are dealing with an environment which could only store integers within the 32-bit signed integer range: [−2^31,  2^31 − 1]. If the numerical value is out of the range of representable values, INT_MAX (2^31 − 1) or INT_MIN (−2^31) is returned.



Example:

    input: "42"
    output: 42

    Input: "   -42"
    Output: -42

    Input: "4193 with words"
    Output: 4193

    Input: "words and 987"
    Output: 0

    Input: "-91283472332"
    Output: -2147483648

#### My answer:

- 解题思路：

    >此题是实现字符串到数字的转换函数，既 *atoi* 函数，这个函数在大一的学习中我们也是经常讨论到，现在就是将这个函数实现出来。

    >不过此题中有不少的规则需要我们去满足来处理特殊情况：

    - 函数首先丢弃尽可能多的空格字符，直到找到第一个非空白字符为止。然后，从这个字符开始，取一个可选的初始加或减号，然后尽可能多的数字数字，并将它们解释为一个数值。

    - 在构成整数的那些字符串之后，字符串可以包含额外的字符，这些字符被忽略，对该函数的行为没有影响。

    - 如果str中的第一个非空白字符序列不是一个有效的整数，或者如果没有这样的序列存在，因为str是空的，或者它只包含空格字符，那么就不会进行转换。

    - 如果不能执行有效的转换，则返回零值。

    针对上述要求我们对不同情况进行分类讨论最后计算出结果即可。

- Code for C++:
```java
        class Solution {
        public:
            int myAtoi(string str) {
                int i=0, result=0, sign=1, temp;

                //忽视所有前导的空格,若 i >= str.length()，则后面的判断都不会发生
                //从而直接跳转至return 语句，返回0值。
                while(i < str.length() && str[i] == ' ')
                    i++;
                
                if(i < str.length() && str[i] == '+')
                    i++;
                //判断是否为负数，默认为正，sign默认为1
                else if(i < str.length() && str[i] == '-')
                {
                    sign = -1; // negative
                    i++;
                }

                //isdigit() 函数：是计算机C(C++)语言中的一个函数，主要用于检查其参数是否为十进制数字字符。
                while(i < str.length() && isdigit(str[i]))
                {
                    temp = result;
                    //逐位进行计算。
                    result = result * 10 + (str[i] - '0');

                    //此步用于满足题目的最后一个要求，既计算的最终结果不能溢出。
                    //若下溢，则返回INT_MIN，上溢则返回INT_MAX。
                    //如果溢出，计算的结果将会发生改变，我们通过temp保存计算前结果，
                    //再与计算后结果进行比较，若值不相等，或者result变为负数则说明溢出，此时再判断上下溢，并返回相应值。
                    if(result < 0 || temp != (result - (str[i] - '0'))/10)
                        return sign == 1 ? INT_MAX : INT_MIN;
                    
                    i++;
                }
                //最后再将符号位乘进去。
                return result * sign;
            }
        };
```
    >注:随时使用 `i < str.length()` 以防止读取数组越界。

    > `int的范围： [-2147483648,2147483647]`。我们针对输入为 "2147483648" 的情况，这时result的补码形式为全0，因为上溢出符号位由0变为1，对应值机器计算出的值是 `-2147483648` ，我们通过`result - (str[i] - '0'))/10` 又可以将值还原，所以 `temp != (result - (str[i] - '0'))/10` 无法判断这种临界情况的上溢，所以我们额外增加一个判断条件，判断result 是否小于 0。
