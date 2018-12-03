---
layout:     post
title:      "LeetCode problems 12"
subtitle:   "First Missing Positive"
date:       2018-11-24 17:32
author:     "Heng"
header-img: "img/弗雷尔卓德2.jpg"
catalog: true
tags:
    - LeetCode
---

>Invictus Gaming——We are the champion！.

---

# First Missing Positive

>Difficulty: `Hard`

### Description:


- Given an unsorted integer array, find the smallest missing positive integer.


----


### 题目描述

- 给定一个未排序的整数数组，查找丢失的最小的正整数。

---

#### Example

    Example 1:

        Input: [1,2,0]
        Output: 3


    Example 2:

        Input: [3,4,-1,1]
        Output: 2


    Example 3:

        Input: [7,8,9,11,12]
        Output: 1


#### Note:

`Your algorithm should run in O(n) time and uses constant extra space.`

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
            int firstMissingPositive(vector<int>& nums) {
                for(int i=0;i<nums.size();i++){
                    //如果当前位置i下出现了比所有数的和还大的值，或者负数值都可以直接忽略，节约空间。
                    //对于符合要求的值就将其与其应该在的位置的数进行交换。
                    //使用wihle循环目的是保证交换到当前位置下的数也要被交换到该处的位置上.
                    //直到当前位置下的数是满足要求的，才进行下一个数的操作。
                    while(nums[i]>0 && nums[i]<=nums.size() && nums[i]!=i+1 && nums[nums[i]-1]!=nums[i]){
                        //nums[nums[i]-1]!=nums[i]) 为了保证不会出现无限互相交换（while循环无法终止）
                        //已经满足要求的数我们也没必要进行交换。
                        int temp=nums[i];
                        //对当前位置马上要被交换到的位置上的数也进行判断，负数及超过数组size的数都可舍弃——直接替换;
                        //符合要求的数保留，并进行交换
                        if(nums[temp-1]>0 && nums[temp-1]<=nums.size()){
                            nums[i]=nums[temp-1];
                            nums[temp-1]=temp;
                        }else{
                            nums[temp-1]=temp;
                            break;
                        }
                    }
                }
                //交换好位置之后，从头开始漏掉的正整数即为所需要的
                for(int i=0;i<nums.size();i++)
                    if(nums[i]!=i+1)   
                        return i+1;
                return nums.size()+1;
            }
        };
    ```
