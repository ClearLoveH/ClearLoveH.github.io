---
layout:     post
title:      "LeetCode problems 14"
subtitle:   "Permutations"
date:       2018-12-8 13:39
author:     "Heng"
header-img: "img/弗雷尔卓德2.jpg"
catalog: true
tags:
    - LeetCode
---

>Invictus Gaming——We are the champion！.

---

# Permutations

>Difficulty: `Hard`

### Description:


- Given a collection of distinct integers, return all possible permutations.

----


### 题目描述

- 给定一组不同的整数，返回所有可能的排列。

---

#### Example


    Input: [1,2,3]
    Output:
    [
    [1,2,3],
    [1,3,2],
    [2,1,3],
    [2,3,1],
    [3,1,2],
    [3,2,1]
    ]

### My answerwer

- 解题思路：

    - 此题其实就是让我们求出一个序列的所有排列情况，按照高中数学我们可以得知，一个大小为n的序列，他的排列加起来共有`n!`种，如果用n个循环嵌套的方式，时间复杂度将来到n的n次方之大，看到这题其实很容易想到应该用递归来做了。
    - 做递归算法，就两个关键点，`递归公式与临界状态`。
    - 高中学习排列组合时的题目都是很有趣的，我解此题就沿用了我在高中解排列题目的思路：
        - 寻找所有的排列一个很易于理解的方式就是——`插空位`。怎么求n个数的所有排列？就是在n-1个数的所有排列中，每一个排列都能产生`n-1+1`个`空位`，我们把第n个数插入到n-1个数的所有排列中的所有的n个空位中，就得到了n个数的所有排列。
        - 换成递归的方式，我们n个数的排列就可以一直递归到最终只有一个数的排列，然后再利用第一个得到的所有排列结果得到两个数时的排列结果，一直到第n个即可。


- Code for C++:
    ```java
        class Solution {
        public:
            vector<vector<int>> permute(vector<int>& nums) {
                vector<vector<int>> answer;
                
                //临界状态：当nums只有一个值或一开始便为空，直接返回nums，这也是递归的临界状态
                if(nums.size() <= 1){
                    answer.push_back(nums);
                }
                else{
                    //我们把给出的序列的第一个数用于插入空位，然后求剩下n-1个数的排列个数（递归）
                    int num = nums[0];
                    nums.erase(nums.begin());
                    //递归
                    vector<vector<int>> afteranswer = permute(nums);
                    //把递归得到结果进行插空处理，注意每一个排列都可以有n-1+1个插空结果，所有共有n*(n-1的所有排列个数)个排列产生。
                    for(int i = 0; i < afteranswer.size(); i++){
                        for(int j = 0; j <= afteranswer[i].size(); j++){
                            vector<int> temp = afteranswer[i];
                            temp.insert(temp.begin() + j, num);
                            answer.push_back(temp);
                        }
                    }
                }
                return answer;
            }
        };
    ```
