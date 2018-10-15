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
    - 这里使用到了C++库中的`upper_bound`函数与`binary_search`函数，算法upper_bound是二分查找（binary search）法的一个版本。它视图在已排序的[first,last)中寻找value。更明确地说，它会返回“在不破坏顺序的情况下，可插入value的最后一个合适的位置。
    - 参考资料：
        - [C++ STL中的Binary search（二分查找）](http://www.cnblogs.com/wkfvawl/p/9475939.html)   
        - [C++中lower_bound函数和upper_bound函数](https://blog.csdn.net/u013475704/article/details/46458723)

- Code for C++:

    ```java
    class Solution {
    public:
        double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
            if(nums1.size()> nums2.size())
                nums1.swap(nums2);
            //如果一个数组为空，则直接求另一个数组的中位数即可
            if(nums1.empty())
                return (nums2.size()% 2) ? nums2[nums2.size()/ 2] : double(nums2[nums2.size()/ 2]+ nums2[nums2.size()/ 2- 1])/ 2;

            int l= min(*nums1.begin(), *nums2.begin()), r= max(*nums1.rbegin(), *nums2.rbegin());int sum_size= nums1.size()+ nums2.size();
            int mid= 0;
            // binary_search函数————在数组中以二分法检索的方式查找，若在数组(要求数组元素非递减)中查找到index元素则真，若查找不到则返回值为假。
            int temp= binary_search(l, r, sum_size/ 2+ 1, nums1, nums2);
            if(sum_size% 2)
                return temp;
            int temp2= binary_search(l, r, sum_size/ 2, nums1, nums2);
            return double(temp+ temp2)/ 2;
        }
        
    private:
        int binary_search(int l, int r, int m, vector<int> nums1, vector<int> nums2){
            while(l< r){
                int temp= (r- l)/ 2+ l;
                //upper_bound(ForwardIter first, ForwardIter last, const _Tp& val)算法返回一个非递减序列[first, last)中的第一个大于值val的位置。
                int p1= upper_bound(nums1.begin(), nums1.end(), temp)- nums1.begin();
                int p2= upper_bound(nums2.begin(), nums2.end(), temp)- nums2.begin();
                if(p1+ p2< m)
                    l= temp+ 1;
                else
                    r= temp;
            }
            return l;
        }
    };
    ```

