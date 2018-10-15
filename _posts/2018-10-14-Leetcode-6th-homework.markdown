---
layout:     post
title:      "LeetCode problems 6"
subtitle:   "Median of Two Sorted Arrays    "
date:       2018-10-14 8:44
author:     "Heng"
header-img: "img/弗雷尔卓德2.jpg"
catalog: true
tags:
    - LeetCode
---

>做了几周算法题目之后，有了点自己的感悟，现在需要慢慢提高所选题目的难度了，给自己点高要求。

---

# Median of Two Sorted Arrays    

>Difficulty: `Hard`

### Description:

There are two sorted arrays nums1 and nums2 of size m and n respectively.

Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).

You may assume nums1 and nums2 cannot be both empty.

----

### 题目描述

- 有两个排序的数组nums1和nums2分别是m和n。
- 找到两个排序数组的中值。总的运行时间复杂度应该是O（log（m+n））。
- nums1和nums2不会同时是空的。

---

#### Example

    nums1 = [1, 3]
    nums2 = [2]

    The median is 2.0
    Example 2:

    nums1 = [1, 2]
    nums2 = [3, 4]

    The median is (2 + 3)/2 = 2.5

### My answer

- 解题思路

    - 此题有一个较为简单的思路就是先进行归并，然后再排序，但是单次归并的复杂度为O(n)，不满足题意，所以另找思路。
    - 这题在上完算法课后来看真的很熟悉，是寻找两个有序数组的中位数，考虑到的是我们讲过的算法，就是`二分查找`无疑了，这个问题是上课时讲过的，对两个数组分别进行二分查找即可，这样我们的时间复杂度就是O(log2(m+n))，满足题目要求。
    - 但是上课我们只是将算法描述出来，并没有具体的去实现，所以现在在这我就需要把这个思路用户代码具体的实现出来。

- Code for C++: