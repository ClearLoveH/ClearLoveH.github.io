---
layout:     post
title:      "LeetCode problems 5"
subtitle:   "3Sum"
date:       2018-10-4 23:09
author:     "Heng"
header-img: "img/恕瑞玛2.jpg"
catalog: true
tags:
    - LeetCode
---

>国庆快乐，比起去看拥挤的人山人海，还是在宿舍呆着悠闲。国庆节的第二篇算法。

---

# 3Sum

>Difficulty: `Medium`

### Description:

- Given an array nums of n integers, are there elements a, b, c in nums such that a + b + c = 0? Find all unique triplets in the array which gives the sum of zero.

### Note:

- The solution set must not contain duplicate triplets.

--- 

### 题目描述：

- 在提供的N个元素的数组中，寻找到所有三个元素和为0的集合。
- 解集不能包含重复的三胞胎。

---

#### Example:

    Given array nums = [-1, 0, 1, 2, -1, -4],

    A solution set is:
    [
        [-1, 0, 1],
        [-1, -1, 2]
    ]

#### My answer:

- 解题思路：

    - 此题我盯着许久了，于是选择在国庆抽了个时间来解决一下。
    - 刚接触LeetCode时看到的第一个问题是 Two Sum，题目大意是让我们在给定数组中寻找到两个数和为给定的target值。此题为Two Sum的升级版，将二维变成了三维，所以解题思路就在Two Sum的基础上进行拓展即可。
    - 这些题目的最直接的一类方法就是几维问题我们就使用几重循环来寻找结果，这种事万金油解法，但是这种解法的

- Code for C++:

    ```c++
        class Solution {
        public:
            vector<vector<int>> threeSum(vector<int>& nums) {
                
            }
        };
    ``` 

>下周预告：`Regular Expression Matching`
