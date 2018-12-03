---
layout:     post
title:      "LeetCode problems 11"
subtitle:   "Longest Valid Parentheses"
date:       2018-11-17 13:06
author:     "Heng"
header-img: "img/艾欧尼亚.jpg"
catalog: true
tags:
    - LeetCode
---

>Invictus Gaming——We are the champion！.

---

# Longest Valid Parentheses

>Difficulty: `Hard`

### Description:


- Given a string containing just the characters '(' and ')', find the length of the longest valid (well-formed) parentheses substring.

----


### 题目描述

- 给定一个只包含字符'('和')'的字符串，查找最长的有效(格式良好)括号子字符串的长度。

---

#### Example

    Example 1:

    Input: "(()"
    Output: 2
    Explanation: The longest valid parentheses substring is "()"


    Example 2:

    Input: ")()())"
    Output: 4
    Explanation: The longest valid parentheses substring is "()()"

### My answer

- 解题思路

    - 此题考查最大子串的问题，但是又限制在圆括号的匹配问题`()`之上，大一C学习过程做过许多类似题目，解决括号匹配问题，判断给定字符串的所有括号是否匹配的问题。
    - 而此题有些不同，需要我们解出最长的且匹配了的括号子串的长度。
    - 当初判断括号是否匹配的问题我们常用解题思路是使用栈来解决，此题同理，不过我们需要把每次pop操作时的状态记录下来，（遇到右括号即pop一次，说明有一对满足）把此时的结果记录下来，目的是需要寻找到最长的括号匹配的子串。
    - 而我们这里使用栈的方式又比较特殊，因为此题没有其他种类的括号，只有圆括号，我们的栈此时没必要把括号push进去，我们把当前字符串的遍历到的字符的序号入栈，这样我们就可以在得到右括号`)`时计算到当前这个子串已经多长了。

- Code for C++:

    ```java
        class Solution {
        public:
            int longestValidParentheses(string s) {
                stack<int> record;
                //index 用于记录当前读取到字符串s的哪个位置
                int result=0,index=-1,tempResult=0;
                for(int i=0;i<s.length();i++){
                    if(s[i]=='(')
                        record.push(i);
                    else{
                        if(record.empty()){
                            index=i;
                        }
                        else{
                            record.pop();
                            if(record.empty())
                                tempResult=i-index;
                            else tempResult=i-record.top();
                            if(tempResult > result)
                                result=tempResult;
                        }
                    }
                }
                return result;
            }
        };
    ```
