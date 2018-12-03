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

    ![](/img/in-post/post-LeetCode/rainwatertrap.png)

----


### 题目描述

- 给定n个非负整数表示高程图，其中每个条形图的宽度为1，计算下雨后它能够截留多少水。如上图所示，我们需要计算的结果即为上图蓝色部分的结果。

---

#### Example

    Example:

    Input: [0,1,0,2,1,0,1,3,2,1,2,1]
    Output: 6


### My answer

- 解题思路：

    - 这道题其实就是让我们算“积水”有多少，解题的思路其实是非常清楚的。
    - 不要被图片所迷惑，我们需要看清其中的本质，解此题一个比较明晰的思路就是，对给出的数组分别从两端进行遍历，对每一列都判断其高度和他`外侧`高度的差值，如果是向内凹的情况，则该列必可以有积水，且积水为外侧列与当前列的高度差值。
    - 为什么要从两端遍历呢？因为如果可以有积水产生，那么此`地形`必定是向内凹的一种地形，即V字地形，我们从两边分别遍历数组，寻找有`坑`的部分，最终两端汇合，即可把所有的`坑`给寻找出来。
    - 对于比较大的`坑`，往往不止是只有一列的，大坑可能由多列组成，那么此坑的第一列的高度对最深列的积水也会产生影响，所以我们每当找到坑的最边缘时，计算一列的积水，还需要把边缘列给传递下去，代码`h[l] = h[l - 1]`就是为了实现这个目的。
    - 最终汇合时，我们还需要进行最后一次判断，汇合列是否有积水，整个图的所有积水最终就计算出来了。


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
