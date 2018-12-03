---
layout:     post
title:      "LeetCode problems 10"
subtitle:   "Substring with Concatenation of All Wordsp"
date:       2018-11-11 11:11
author:     "Heng"
header-img: "img/艾欧尼亚.jpg"
catalog: true
tags:
    - LeetCode
---

>Invictus Gaming——We are the champion！.

---

# Substring with Concatenation of All Words

>Difficulty: `Hard`

### Description:


- You are given a string, s, and a list of words, words, that are all of the same length. Find all starting indices of substring(s) in s that is a concatenation of each word in words exactly once and without any intervening characters.

----


### 题目描述

- 给你一个字符串，s，和一个单词列表，words，且每个单词都是相同的长度。查找s中所有子字符串的起始索引，该索引将单词中的每个单词精确地连接在一起，而不包含任何插入字符。

---

#### Example

    Example 1:

    Input:
    s = "barfoothefoobarman",
    words = ["foo","bar"]
    Output: [0,9]
    Explanation: Substrings starting at index 0 and 9 are "barfoor" and "foobar" respectively.
    The output order does not matter, returning [9,0] is fine too.

    Example 2:

    Input:
    s = "wordgoodstudentgoodword",
    words = ["word","student"]
    Output: []

### My answer

- 解题思路

    - 此题思路比较清晰，因为题目已经给出所有需要连接后查询的单词都是相同长度的，我们先从总串s头字母开始查询`(i = 0-->s.length-wordCount * wordlen)`——（减去`wordCount * wordlen`是为了因为s后面的`剩余长度小于我们需要查询的连接子串长度的部分`可以直接忽略）。
    - 因为每个word length固定，所以第二重循环我们就对`每length`个字母作为一个查询的word`(j = 0; j < wordCount; j++)`，查询是否在words数组里，直到查出j个连续的这样的word，返回此时i(s中的index)。
    - 解题需使用到`substr`函数：substr是C++语言函数，主要功能是复制子字符串，要求从指定位置开始，并具有指定的长度。如果没有指定长度_Count或_Count+_Off超出了源字符串的长度，则子字符串将延续到源字符串的结尾。

- Code for C++:

    ```java
        class Solution {
        public:
            vector<int> findSubstring(string s, vector<string>& words) {
                // word frequency
                unordered_map<string, int> wrodFreq; 
                vector<int>                answer;
                int                        slen;
                // word count and word len 
                int                        wordCount, wordlen; 
                
                if (words.empty())
                    return answer;
                for (string word : words)
                    wrodFreq[word]++;
                
                slen = s.length();
                //需要连接的单词数
                wordCount = words.size();
                wordlen = words[0].size();
                int i = 0, j = 0;
                for (i = 0; i < slen - wordCount * wordlen + 1; i++) {
                    unordered_map<string, int> seen;
                    for (j = 0; j < wordCount; j++) {
                        //对每Wordlen个字母分别进行查询
                        string word = s.substr(i + j * wordlen, wordlen);
                        //如果未查询到对应单词，则break，否则进行下一个word的查询，j++
                        if (wrodFreq.find(word) == wrodFreq.end())
                            break;
                        //记录每个word出现的次数，不能大于words数组中出现的频率。
                        seen[word]++;
                        if (seen[word] > wrodFreq[word])
                            break;
                    }
                    //若j循环完全，证明words中所有的word都被连接上了，此时j = wordCount，则该子串有效
                    if (j == wordCount) 
                        answer.push_back(i);
                }
                return answer;
            }
        };
    ```
