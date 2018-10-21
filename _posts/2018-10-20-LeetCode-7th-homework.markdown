---
layout:     post
title:      "LeetCode problems 6"
subtitle:   "Regular Expression Matching"
date:       2018-10-20 12:38
author:     "Heng"
header-img: "img/弗雷尔卓德2.jpg"
catalog: true
tags:
    - LeetCode
---

>Royal may give up now..

---

# Regular Expression Matching   

>Difficulty: `Hard`

### Description:


- Given an input string (s) and a pattern (p), implement regular expression matching with support for '.' and '*'.

        '.' Matches any single character.
        '*' Matches zero or more of the preceding element.

- The matching should cover the entire input string (not partial).

----


### Node

- s could be empty and contains only lowercase letters a-z.
- p could be empty and contains only lowercase letters a-z, and characters like . or *.

---

### 题目描述

- 给定一个输入字符串和一个模式，实现正则表达式匹配，并支持"."和"*"。
    - "."匹配任何单个字符。
    - "*"匹配前一个元素的0个或多个。
- 匹配应该覆盖整个输入字符串(而不是部分)。

---

#### Example

Example 1:

    Input:
    s = "aa"
    p = "a"
    Output: false
    Explanation: "a" does not match the entire string "aa".

Example 2:

    Input:
    s = "aa"
    p = "a*"
    Output: true
    Explanation: '*' means zero or more of the precedeng element, 'a'. Therefore, by repeating 'a' once, it becomes "aa".

Example 3:

    Input:
    s = "ab"
    p = ".*"
    Output: true
    Explanation: ".*" means "zero or more (*) of any character (.)".

Example 4:

    Input:
    s = "aab"
    p = "c*a*b"
    Output: true
    Explanation: c can be repeated 0 times, a can be repeated 1 time. Therefore it matches "aab".

Example 5:

    Input:
    s = "mississippi"
    p = "mis*is*p*."
    Output: false

### My answer

- 解题思路

    - 此题有一个较为简单的思路就是先进行归并，然后再排序，但是单次归并的复杂度为O(n)，不满足题意，所以另找思路。
    - 这题在上完算法课后来看真的很熟悉，是寻找两个有序数组的中位数，考虑到的是我们讲过的算法，就是`二分查找`无疑了，这个问题是上课时讲过的，对两个数组分别进行二分查找即可，这样我们的时间复杂度就是O(log2(m+n))，满足题目要求。
    - 但是上课我们只是将算法描述出来，并没有具体的去实现，所以现在在这我就需要把这个思路用户代码具体的实现出来。
    - 这里使用到了C++库中的`upper_bound`函数与`binary_search`函数，算法upper_bound是二分查找（binary search）法的一个版本。它视图在已排序的[first,last)中寻找value。更明确地说，它会返回“在不破坏顺序的情况下，可插入value的最后一个合适的位置。
    - 参考资料：
        - [C++ STL中的Binary search（二分查找）](http://www.cnblogs.com/wkfvawl/p/9475939.html)   
        - [C++中lower_bound函数和upper_bound函数](https://blog.csdn.net/u013475704/article/details/46458723)

- Code for C++:
