---
layout:     post
title:      "LeetCode problems 1"
subtitle:   "Add Two Numbers"
date:       2018-9-6 15:30
author:     "Heng"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - LeetCode
---

>作为我的博客的第一篇文章。本文选取了LeetCode上一个中等难度的问题来进行算法的设计。

---

#  Add two numbers 
>Difficulty: Medium

### Description: 

-   You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.
<br>You may assume the two numbers do not contain any leading zero, except the number 0 itself.

    
- 给您两个非空链表，表示两个非负整数。这些数字以相反的顺序存储，每个节点都包含一个数字。将两个数字相加，并将其作为链表返回。<br>你可以假设这两个数字不包含任何前导零，除了数字0本身。

#### Example:

    Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
    Output: 7 -> 0 -> 8
    Explanation: 342 + 465 = 807.

#### My answer:


- 解题思路：

    1. 因为此题是将数字用倒序链表来表示，链表头是数字的最低位，相比与链表头是数字最高位的表示方法，这种方法减少了许多计算时的难度。
    2. 我们在做数的加法运算时，都是从最低位开始加，满十则向前进一，在此题我们也是贯彻这种计算思路不变，不过难点在于计算结果也需要用链表表示。
    3. 申请一个新的结果链表用于存储结果，从两个计算链表的头节点开始相加，计算结果存入结果链表的头节点。
    4. 申请一个变量carry=0，用于存储进位。若当前位的相加结果result >= 10，result = result % 10，carry = 1。将result值存入链表当前节点中，分别访问三条链表的下一个节点（list = list -> next）。
    5. 重复第4步直至两个计算数链表都访问完(list1->next != NULL && list2-> next != NULL)。
    6. 最后，将两个运算数中，位数长的那个数剩下的位数加入到result链表中，return。
    7. 需要注意的是，这里我们可以用%求余，也可以用减法-10的方式来获取每一位计算结果的余数，选择%而不用-是因为取余%的运算时间开销更少。

- Code for C++:
```c++
        /**
         * Definition for singly-linked list.
         * struct ListNode {
         *     int val;
         *     ListNode *next;
         *     ListNode(int x) : val(x), next(NULL) {}
         * };
         */
        class Solution {
            public:
                ListNode* addTwoNumbers(ListNode* list1, ListNode* list2) {
                int carry = 0;
                int currentCarrySum = list1->val + list2->val + carry;
                if (currentCarrySum >= 10) {
                    currentCarrySum = currentCarrySum % 10;
                    carry = 1;
                }
                else carry = 0;
                
                ListNode* result = new ListNode(currentCarrySum);
                ListNode* pointer = result;
                list1 = list1->next;
                list2 = list2->next;

                //计算两位运算数的前面共同位数的计算
                
                while (list1 != NULL && list2 != NULL) {
                    int currentCarrySum = list1->val + list2->val + carry;
                    if (currentCarrySum >= 10) {
                        currentCarrySum = currentCarrySum % 10;
                        carry = 1;
                    }
                    else carry = 0;

                    pointer->next = new ListNode(currentCarrySum);
                    pointer = pointer->next;
                    list1 = list1->next;
                    list2 = list2->next;
                }

                //将运算数list1和list2中较长的那个数剩下的位数进行处理

                while (list1 != NULL) {
                    if (list1->val + carry == 10) {
                        pointer->next = new ListNode(0);
                        carry = 1;
                    }
                    else {
                        pointer->next = new ListNode(list1->val + carry);
                        carry = 0;
                    }
                    pointer = pointer->next;
                    list1 = list1->next;
                }
                while (list2 != NULL) {
                    if (list2->val + carry == 10) {
                        pointer->next = new ListNode(0);
                        carry = 1;
                    }
                    else {
                        pointer->next = new ListNode(list2->val + carry);
                        carry = 0;
                    }
                    pointer = pointer->next;
                    list2 = list2->next;
                }

                //若最终进位仍位1，最后在结果链表再添上一位

                if (carry == 1) 
                    pointer->next = new ListNode(1);

                return result;
            }
        };
```

