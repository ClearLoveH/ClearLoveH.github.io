---
layout:     post
title:      "LeetCode problems 4"
subtitle:   "Swap Nodes in Pairs"
date:       2018-9-29 15:27
author:     "Heng"
header-img: "img/恕瑞玛3.jpg"
catalog: true
tags:
    - LeetCode
---

>国庆快乐，小长假去哪里玩，不如在家写写算法。

---

# Swap Nodes in Pairs

>Difficulty: Medium

### Description:

- Given a linked list, swap every two adjacent nodes and return its head.

### Note:

- Your algorithm should use only constant extra space.

- You may not modify the values in the list's nodes, only nodes itself may be changed.

--- 

### 题目描述：

- 给定一个链表，交换每两个相邻的节点并返回它的头部。
- 你的算法应该只使用`恒定的额外空间`。
- 您可能不会修改列表节点中的值，只有节点本身可能会被更改。

---

#### Example:

    Given 1->2->3->4
    you should return the list as 2->1->4->3.

#### My answer:

- 解题思路：

    - 许久没有做链表题目了，于此选取了一个看起来比较有趣的链表中节点交换的问题。题目虽然不长，但是也有我们需要思考的地方。
    - 此题无疑是要遍历链表全部节点的，为了减少算法复杂度，我们需要在遍历过程中就进行节点的交换。
    - 每对节点交换完成后需继续下一对的交换。
    - 若链表共有奇数个节点，最后必然会剩下一个，这个该如何处理是我们再写遍历循环时需要考虑的条件。
    `(考虑在遍历的循环条件中就判断出剩下节点是否只剩一个)`
    - 题目相对简单，于是我们在算法的复杂度上需多花些功夫。
    - 还有个注意点就是题目的最后一个要求——`不允许改变节点的值`，题目应该是不允许我们钻空子直接更改两个节点的value，而是需要去直接改变节点的链接关系。

- Code for C++:

    - 节点的定义：
    ```c++
        struct ListNode {
            int val;
            ListNode *next;
            ListNode(int x) : val(x), next(NULL) {}
        };    
    ```

    - 我的结果：
    ```c++
        class Solution {
                public:
                ListNode* swapPairs(ListNode* head) {       
                    if(head==NULL)
                        return NULL;
                    ListNode *temp = head , *temp1;   

                    //(head && head->next) 判断条件用于判断是否到达最后的节点

                    if(head && head->next){ 
                        head=head->next;
                        temp1=head->next;
                        head->next=temp;
                        
                        //使用递推方法，引用函数本身，以达到持续访问玩所有节点的目的。

                        temp->next=swapPairs(temp1);
                    }
                return head;
            }
        };
    ``` 

>注：此题看似简单，实则思路很多，而对于处理完第一对节点之后该怎么继续下去，我在本题目的思路是使用递推方法，调用函数自身，` temp->next=swapPairs(temp->next)` 将交换结束前的第三个节点作为后续剩下节点构成的链表的头节点，调用此方法，从而达到向后继续遍历的目的。
