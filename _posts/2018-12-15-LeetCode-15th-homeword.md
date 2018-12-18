---
layout:     post
title:      "LeetCode problems 15"
subtitle:   "Jump Game II"
date:       2018-12-15 19:22
author:     "Heng"
header-img: "img/弗雷尔卓德2.jpg"
catalog: true
tags:
    - LeetCode
---

>广州入冬了，本学期最后一次的LeetCode博客。

---


# Jump Game II

>Difficulty: `Hard`

### Description:

- Given an array of non-negative integers, you are initially positioned at the first index of the array.
- Each element in the array represents your maximum jump length at that position.
- Your goal is to reach the last index in the minimum number of jumps.

----


### 题目描述

- 给定一个非负整数数组，您最初定位在数组的第一个索引处。
- 数组中的每个元素表示该位置的最大跳转长度。
- 您的目标是在最少的跳跃次数中达到最后一个索引。

---

#### Example

    Input: [2,3,1,1,4]
    Output: 2
    Explanation: The minimum number of jumps to reach the last index is 2.
        Jump 1 step from index 0 to 1, then 3 steps to the last index.

#### Note:

You can assume that you can always reach the last index.

### My answerwer

- 解题思路：

    - 理解了题目之后想到贪心算法，贪心来解此题似乎是可行的，每一次选择都选择下一步可以到达的点中，jump length中最大的点（该点位置加该点的可跳长度最大），而这个length并不是每一跳一定要走的长度，是每一步可走的最大长度，没有说一定要正好跳到终点处，即是可以选择的，我们可以用这种思路来解决此问题。
    - 这种贪心策略时间复杂度为n方，思路简单但是时间复杂度还是略大。

- Code for C++:
    ```java
        class Solution {
            public int jump(int[] nums) {
                int sum_length = nums.size() - 1;
                int jumps = 0;
                int i = 0;
                while(i + nums[i] < sum_length){
                    int max = 0;
                    int next_index = i + 1;
                    for(int j = next_index;j < i + num[i];j++){
                        if(num[j] + i > max){
                            max = num[j] + i;
                            next_index = j;
                        }
                    }
                    i = next_index;
                    jumps++;
                }
                return ++jumps;
            }
        }
    ```


