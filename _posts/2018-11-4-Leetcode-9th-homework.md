---
layout:     post
title:      "LeetCode problems 9"
subtitle:   "Reverse Nodes in k-Group"
date:       2018-11-4 9:25
author:     "Heng"
header-img: "img/弗雷尔卓德2.jpg"
catalog: true
tags:
    - LeetCode
---

>Invictus Gaming——We are the champion！.

---

# Reverse Nodes in k-Group 

>Difficulty: `Hard`

### Description:


- Given a linked list, reverse the nodes of a linked list k at a time and return its modified list.

- k is a positive integer and is less than or equal to the length of the linked list. If the number of nodes is not a multiple of k then left-out nodes in the end should remain as it is.

----


### 题目描述

- 给定一个链表，每次反转链表k的节点并返回其修改后的链表。
- k是一个正整数，小于或等于链表的长度。如果节点数不是k的倍数，那么最后应该保留原来的节点。

---

#### Example

    Example:

    Given this linked list: 1->2->3->4->5

    For k = 2, you should return: 2->1->4->3->5

    For k = 3, you should return: 3->2->1->4->5

### My answer

- 解题思路

    - 此题思路比较简单，我的解题思路就是对链表每K个节点分别进行倒序，操作完之后进行下一步K节点的倒序，一直到最后剩下的小于K的剩余节点保持原序不变，此题就解出来了，题目理解起来难度不大，但是非常考验我们对节点交换时的操作，注意节点的next指针也正确转移即可，。
    - 重点就是K个节点的逆序怎么去实现了，这里有两种大的实现方法：
        - 使用栈结构，把每K个节点压入栈中，再依次出栈，这样实现起来非常简单。
        - 不使用栈结构，直接在原链表上修改，就是对链表进行遍历，也是我这里使用的方法。

- Code for C++:

    ```java
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
            ListNode* reverseKGroup(ListNode* head, int k) {
                if (head==NULL || head->next==NULL || k==1)
                    return head;
                int length=0;
                ListNode* count = head;
                while(count)
                {
                    count = count->next;
                    length++;
                }
                if(length<k)
                    return head;
                //计算出共需要逆序几轮
                int round = length / k;
                ListNode* dummy = new ListNode(-1);
                dummy->next=head;
                ListNode* pNode=head;
                ListNode* preNode=dummy;
                ListNode* pNext = pNode->next;
                for (int turn=0;turn<round;turn++){
                    ListNode* firstK = pNode;
                    for (int i=1;i<k;i++){
                        ListNode* temp = pNext->next;
                        pNext->next = pNode;
                        pNode = pNext;
                        pNext = temp;
                    }
                    preNode->next = pNode;
                    firstK->next = pNext;
                    if(pNext==NULL || pNext->next==NULL)
                        return dummy->next;
                    pNode = pNext;
                    pNext = pNext->next;
                    preNode = firstK;
                    firstK = pNode;
                }
                return dummy->next;
            }
        };
    ```
