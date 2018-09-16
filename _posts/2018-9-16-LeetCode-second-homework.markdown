---
layout:     post
title:      "LeetCode problems 2"
subtitle:   "String to Integer (atoi)"
date:       2018-9-16 13:42
author:     "Heng"
header-img: "img/post-bg-2015.jpg"
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

    - 实现将一个字符串转换成整数的方法。
    - 函数首先丢弃尽可能多的空格字符，直到找到第一个非空白字符为止。然后，从这个字符开始，取一个可选的初始加或减号，然后尽可能多的数字数字，并将它们解释为一个数值。
    - 在构成整数的那些字符串之后，字符串可以包含额外的字符，这些字符被忽略，对该函数的行为没有影响。
    - 如果str中的第一个非空白字符序列不是一个有效的整数，或者如果没有这样的序列存在，因为str是空的，或者它只包含空格字符，那么就不会进行转换。
    - 如果不能执行有效的转换，则返回零值。

#### Note:

- Only the space character ' ' is considered as whitespace character.
- Assume we are dealing with an environment which could only store integers within the 32-bit signed integer range: [−2^31,  2^31 − 1]. If the numerical value is out of the range of representable values, INT_MAX (2^31 − 1) or INT_MIN (−2^31) is returned.



Example:

    input: "42"
    output: 42