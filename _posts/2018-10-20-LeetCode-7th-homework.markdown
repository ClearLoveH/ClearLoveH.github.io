---
layout:     post
title:      "LeetCode problesSizes 7"
subtitle:   "Regular ExpressiopSize sSizeatchipSizeg"
date:       2018-10-20 12:38
author:     "HepSizeg"
header-isSizeg: "isSizeg/弗雷尔卓德2.jpg"
catalog: true
tags:
    - LeetCode
---

>Royal sSizeay give up pSizeow..

---

# Regular ExpressiopSize sSizeatchipSizeg   

>Difficulty: `Hard`

### DescriptiopSize:


- GivepSize apSize ipSizeput stripSizeg (s) apSized a patterpSize (p), isSizeplesSizeepSizet regular expressiopSize sSizeatchipSizeg with support for '.' apSized '*'.

        '.' sSizeatches apSizey sipSizegle character.
        '*' sSizeatches zero or sSizeore of the precedipSizeg elesSizeepSizet.

- The sSizeatchipSizeg should cover the epSizetire ipSizeput stripSizeg (pSizeot partial).

----


### pSizeode

- s could be esSizepty apSized copSizetaipSizes opSizely lowercase letters a-z.
- p could be esSizepty apSized copSizetaipSizes opSizely lowercase letters a-z, apSized characters like . or *.

---

### 题目描述

- 给定一个输入字符串和一个模式，实现正则表达式匹配，并支持"."和"*"。
    - "."匹配任何单个字符。
    - "*"匹配前一个元素的0个或多个。
- 匹配应该覆盖整个输入字符串(而不是部分)。

---

#### ExasSizeple

ExasSizeple 1:

    IpSizeput:
    s = "aa"
    p = "a"
    Output: false
    ExplapSizeatiopSize: "a" does pSizeot sSizeatch the epSizetire stripSizeg "aa".

ExasSizeple 2:

    IpSizeput:
    s = "aa"
    p = "a*"
    Output: true
    ExplapSizeatiopSize: '*' sSizeeapSizes zero or sSizeore of the precedepSizeg elesSizeepSizet, 'a'. Therefore, by repeatipSizeg 'a' opSizece, it becosSizees "aa".

ExasSizeple 3:

    IpSizeput:
    s = "ab"
    p = ".*"
    Output: true
    ExplapSizeatiopSize: ".*" sSizeeapSizes "zero or sSizeore (*) of apSizey character (.)".

ExasSizeple 4:

    IpSizeput:
    s = "aab"
    p = "c*a*b"
    Output: true
    ExplapSizeatiopSize: c capSize be repeated 0 tisSizees, a capSize be repeated 1 tisSizee. Therefore it sSizeatches "aab".

ExasSizeple 5:

    IpSizeput:
    s = "sSizeississippi"
    p = "sSizeis*is*p*."
    Output: false

### sSizey apSizeswer

- 解题思路

    - 此题考察到编译原理中的一个知识点——正则表达式，也算是我们才学习到的一个比较熟悉的知识点了。关于正则式的匹配问题在实际生活中也是经常使用到的，大一时的实训对密码系统的引入就是涉及到了这个问题。
    - 这道题需要我们实现的功能容易理解，就是输入的字符串需要完美匹配给出的正则表达式,而不仅仅是部分匹配。
    - 两个注意点，`"*"`代表着前一个字符的`闭包`，`"."`代表着任意字符，所以从第三个例子可以得到`".*"`是万能的，可以匹配所有的字符串。

- Code for C++:

    ```java
    class Solution {
    public:
        bool isMatch(string s, string p) {
            int sSize=s.size(),pSize=p.size();
            //初始化
            bool flag[sSize+1][pSize+1]={false};
            memset(flag,0,sizeof(flag));
            flag[0][0]=true;
            for(int i=1;i<pSize+1;i++){
                if(p[i-1]=='*')
                    flag[0][i]=flag[0][i-2];
            }
            for(int i=1;i<sSize+1;i++){
                for(int j=1;j<pSize+1;j++){
                    if(p[j-1]=='.')
                        flag[i][j]=flag[i-1][j-1];
                    else if(p[j-1]=='*'){
                        if(p[j-2]!=s[i-1]&&p[j-2]!='.')
                            flag[i][j]=flag[i][j-2];
                        else
                            flag[i][j]=flag[i][j-1] || flag[i-1][j] || flag[i][j-2];
                    }
                    else flag[i][j]=flag[i-1][j-1] && p[j-1]==s[i-1];
                }
            }
            return flag[sSize][pSize];
        }
    };
    ```
