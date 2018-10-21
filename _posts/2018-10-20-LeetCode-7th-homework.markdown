---
layout:     post
title:      "LeetCode problems 7"
subtitle:   "Regular Expression Matching"
date:       2018-10-20 12:38
author:     "Heng"
header-img: "img/弗雷尔卓德2.jpg"
catalog: true
tags:
    - LeetCode
---

>Royal may give up now..

---

# Regular Expression Matching   

>Difficulty: `Hard`

### Description:


- Given an input string (s) and a pattern (p), implement regular expression matching with support for '.' and '*'.

        '.' Matches any single character.
        '*' Matches zero or more of the preceding element.

- The matching should cover the entire input string (not partial).

----


### Node

- s could be empty and contains only lowercase letters a-z.
- p could be empty and contains only lowercase letters a-z, and characters like . or *.

---

### 题目描述

- 给定一个输入字符串和一个模式，实现正则表达式匹配，并支持"."和"*"。
    - "."匹配任何单个字符。
    - "*"匹配前一个元素的0个或多个。
- 匹配应该覆盖整个输入字符串(而不是部分)。

---

#### Example

Example 1:

    Input:
    s = "aa"
    p = "a"
    Output: false
    Explanation: "a" does not match the entire string "aa".

Example 2:

    Input:
    s = "aa"
    p = "a*"
    Output: true
    Explanation: '*' means zero or more of the precedeng element, 'a'. Therefore, by repeating 'a' once, it becomes "aa".

Example 3:

    Input:
    s = "ab"
    p = ".*"
    Output: true
    Explanation: ".*" means "zero or more (*) of any character (.)".

Example 4:

    Input:
    s = "aab"
    p = "c*a*b"
    Output: true
    Explanation: c can be repeated 0 times, a can be repeated 1 time. Therefore it matches "aab".

Example 5:

    Input:
    s = "mississippi"
    p = "mis*is*p*."
    Output: false

### My answer

- 解题思路

    - 此题考察到编译原理中的一个知识点——正则表达式，也算是我们才学习到的一个比较熟悉的知识点了。关于正则式的匹配问题在实际生活中也是经常使用到的，大一时的实训对密码系统的引入就是涉及到了这个问题。
    - 这道题需要我们实现的功能容易理解，就是输入的字符串需要完美匹配给出的正则表达式,而不仅仅是部分匹配。
    - 两个注意点，`"*"`代表着前一个字符的`闭包`，`"."`代表着任意字符，所以从第三个例子可以得到`".*"`是万能的，可以匹配所有的字符串。

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
                        else flag[i][j]=flag[i][j-1] || flag[i-1][j] || flag[i][j-2];
                    }
                    else flag[i][j]=flag[i-1][j-1] && p[j-1]==s[i-1];
                }
            }
            return flag[s_Size][p_Size];
        }
    };
    ```
