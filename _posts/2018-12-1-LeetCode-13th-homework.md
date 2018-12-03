---
layout:     post
title:      "LeetCode problems 13"
subtitle:   "Trapping Rain Water"
date:       2018-12-1 10:40
author:     "Heng"
header-img: "img/弗雷尔卓德2.jpg"
catalog: true
tags:
    - LeetCode
---

>Invictus Gaming——We are the champion！.

---

# Trapping Rain Water

>Difficulty: `Hard`

### Description:


- Given n non-negative integers representing an elevation map where the width of each bar is 1, compute how much water it is able to trap after raining.


----


### 题目描述

- 给定n个非负整数表示高程图，其中每个条形图的宽度为1，计算下雨后它能够截留多少水。

---

#### Example

    Example:

    Input: [0,1,0,2,1,0,1,3,2,1,2,1]
    Output: 6


### My answer

- 解题思路：

    - 解此题我们很容易想到先把给出的数组进行排序，然后从头开始寻找缺失的正整数即为答案，但是我们常用的排序算法时间复杂度如插入排序、选择排序、冒泡排序、快速排序，他们的平均时间复杂度最好也在O(N*logN)，是不满足本题的要求的。所以我们思考到一个算法——桶排序。  
    - 本题解题思路利用到的是`桶排序`的原理：

        - 桶排序 (Bucket sort)或所谓的箱排序，是一个排序算法，工作的原理是将数组分到有限数量的桶子里。每个桶子再个别排序（有可能再使用别的排序算法或是以递归方式继续使用桶排序进行排序）。桶排序是鸽巢排序的一种归纳结果。当要被排序的数组内的数值是均匀分配的时候，桶排序使用线性时间O(N)。但桶排序并不是 比较排序，他不受到 O(N*logN)下限的影响。

        - 桶排序的缺点就是空间消耗过大，所以此题我们也需要注意这个方面的问题。
    - 此题使用的是桶排序的一个适应本题的“改良版”版本，是将所有符合要求的待排数（小于0或者值大于数组size的数都不符合要求，直接替换掉）通过交换放在他们应该处于的位置上，然后在从头遍历改序后的数组，找到缺漏的那个位置即可，忽略掉非正数及超过数组size的数是因为本题根本用不到，我们无需浪费空间在负数的处理上。
    - 按照这种思路，我们最终会把值为1的数放在i=1-1=0的位置上，值为5的放在i=5-1的位置上，找到最终某个位置不符合这样要求的，就是我们需要的缺漏的那个值。


- Code for C++:

    ```java
        class Solution {
        public:
            int trap(vector<int>& height) {

                if(height.size()==0)
                    return 0;  

                vector<int> h;

                //对原本的数组扩展，两端都添加一个0，以保证后面算法的临界情况数组下表不会溢出
                h.push_back(0);
                for(int i =0; i< height.size(); i++) 
                    h.push_back(height[i]);
                h.push_back(0);

                //初始化处理后，分别从数组两端开始逐个分析
                int l = 1;
                int r = h.size() - 2;
                int sum = 0;
                while (l < r) {
                    if (h[l] < h[r]) {
                        if (h[l] < h[l - 1]) {
                            sum += h[l - 1] - h[l];
                            h[l] = h[l - 1];
                        }
                        l++;
                    }
                    else {
                        if (h[r] < h[r + 1]) {
                            sum += h[r + 1] - h[r];
                            h[r] = h[r + 1];
                        }
                        r--;
                    }
                }
                //最终遍历到l r 相等时，进行最后的判断
                if (l == r && h[l] < min(h[l - 1], h[l + 1])) 
                    sum += min(h[l - 1], h[l + 1]) - h[l];
                return sum;
            }
        };
    ```
