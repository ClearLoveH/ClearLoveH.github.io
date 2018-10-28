---
layout:     post
title:      "LeetCode problems 8"
subtitle:   "Merge k Sorted Lists"
date:       2018-10-28 12:05
author:     "Heng"
header-img: "img/弗雷尔卓德2.jpg"
catalog: true
tags:
    - LeetCode
---

>Invictus Gaming never give up.

---

# Merge k Sorted Lists    

>Difficulty: `Hard`

### Description:


- Merge k sorted linked lists and return it as one sorted list. Analyze and describe its complexity.


----



### 题目描述

- 合并排序后的链表，并将其作为一个排序后的链表返回。分析并描述其复杂性。

---

#### Example

    Example:

    Input:
    [
    1->4->5,
    1->3->4,
    2->6
    ]
    Output: 1->1->2->3->4->4->5->6

### My answer

- 解题思路

    - 

- Code for C++:

    ```java
    class Solution {
    public:
        bool isMatch(string s, string p) {
            int s_Size=s.size(),p_Size=p.size();
            //初始化
            bool flag[s_Size+1][p_Size+1]={false};
            memset(flag,0,sizeof(flag));
            //flag[0][0]代表着空字符串与空模式匹配，设为true
            flag[0][0]=true;

            for(int i=1;i<p_Size+1;i++){
                if(p[i-1]=='*')
                    flag[0][i]=flag[0][i-2];
            }
            
            for(int i=1;i<s_Size+1;i++){
                for(int j=1;j<p_Size+1;j++){
                    if(p[j-1]=='.')
                        flag[i][j]=flag[i-1][j-1];
                    else if(p[j-1]=='*'){
                        if(p[j-2]!=s[i-1] && p[j-2]!='.')
                            flag[i][j]=flag[i][j-2];
                        else flag[i][j] = flag[i][j-1] || flag[i-1][j] || flag[i][j-2];
                    }
                    else flag[i][j]=flag[i-1][j-1] && p[j-1]==s[i-1];
                }
            }
            return flag[s_Size][p_Size];
        }
    };
    ```
