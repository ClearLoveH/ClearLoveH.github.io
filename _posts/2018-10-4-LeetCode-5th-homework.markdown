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
    - 刚接触LeetCode时看到的第一个问题是 Two Sum，题目大意是让我们在给定数组中寻找到两个数和为给定的target值。此题为Two Sum的升级版，将二维问题升成了三维，所以解题思路就在Two Sum的基础上进行拓展即可。
    - 这些题目的最直接的一类方法就是几维问题我们就使用几重循环来寻找结果，这种事万金油解法，但是这种解法的时间复杂度也是最高的，为此我们就需要在减少时间复杂度上下功夫。
    - 还有一个注意点就是最终不能出现重复的结果，而题目所给的数组是允许多个相同元素出现的并且数给定组内元素是无序的，所以我们先事先排好序。

- Code for C++:

    ```java
        class Solution {
        public:
            vector<vector<int>> threeSum(vector<int>& nums) {
                vector<vector<int>> result;
                if(nums.size() <= 2)
                    return result;
                //调用C标准库中的排序函数来进行排序
                sort(nums.begin(),nums.end());
                //减2是为了省去多余的开销，因为我们遍历寻找是三个数的集合，最后两位无需再单独访问。
                for(int i = 0; i < nums.size() - 2; i++){
                    int front = i + 1;
                    int back = nums.size() - 1;
                    int a = nums[i];
                    //若出现重复元素，该元素的第一个已经查询并输入了所有的结果，所有后面的重复情况皆可省去。
                    //但是反过来不可，即不可先将重复元素舍去再寻找结果，如题目所给example中的`[-1,-1,2]`就会因此被抹去。
                    if( i > 0 && a == nums[i-1])
                        continue;
                    while(front < back){
                        int b = nums[front];
                        int c = nums[back];
                        
                        if(a+b+c == 0){
                            result.push_back({a,b,c});     
                            //两个while循环在这里是为了去除相同元素对最终结果的影响，同时前进访问数组的下一个可能结果。
                            //因为题目要求不能出现重复的结果，我们在确定第一个数a(num[i])的情况下，
                            //要出现不同与当前查询到的结果，front和back必然需要同时改变才行，
                            //所以下面的两个while第一次是必然执行的，继续探索是否还有在a相同的情况下，3Sum为0的其他结果。
                            while(front < back && b == nums[front])
                                front++;    
                            while(front < back && c == nums[back])
                                back--;
                        }
                        else if( a+b+c < 0)
                            front++;
                        else back--;
                    }
                }
                return result;
            }
        };
    ``` 

>下周预告：`Regular Expression Matching`
