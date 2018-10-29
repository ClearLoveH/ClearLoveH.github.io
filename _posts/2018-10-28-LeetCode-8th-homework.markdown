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

    - 本题需要我们将提供的几个有序链表合并成一个新的有序链表，具体实现思路有两种：
        - Merge Sort，归并算法。注意如果我们逐一合并，会导致时间复杂度太大——O(n^2)，不合要求，所以只能两两归并来解决此题。
        - 堆排序，这种算法我之前使用的比较少，就通过自己学习了解该方法，尝试将其应用于解决此题。
            - [关于STL的priorit_queue](https://www.cnblogs.com/Deribs4/p/5657746.html)
            - priorit_queue是一种优先级队列，`他的底层是用堆来实现的`，每次为队列push一个新的元素，priorit_queue都会动态的根据优先级来进行排序，所以此题于此显得非常简单了，我们只需要将所有节点push进优先级队列中，即可自动完成排序。
                - 如果不使用priorit_queue也能自己实现一个最大堆或最小堆，同时的方法去使用即可。
            - 注意priorit_queue默认是从大至小排序的，我们只需要修改一下队列排序的比较方式即可。
            - 注意对空输入的处理
        - 经过两种算法权衡之后，优先级队列使用耗时很大，所以最后还是使用了二分合并。
        
- Code for C++:

    - 归并：每次合并后都返回合并链表的头节点，最终把list中所有的链表都合并完成，返回头节点即可。
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
                ListNode* mergeKLists(vector<ListNode*>& lists) {
                    if(lists.size()==0)
                        return NULL;
                    //分治每次循环都对n除以2，知道n==1为止，归并完成
                    int n = lists.size();
                    while (n > 1) {    
                        int k = (n + 1) / 2;    
                        for (int i = 0; i < n / 2; i++)    
                            lists[i] = mergeTwoLists(lists[i], lists[i + k]);    
                        n = k;    
                    }    
                    return lists[0]; 
                    
                }
                //在mergeTwoLists函数里我们常规的对两个链表归并即可。
                ListNode *mergeTwoLists(ListNode *list1,ListNode *list2){
                    if(list1==NULL)
                        return list2;
                    if(list2==NULL)
                        return list1;
                    //头节点result此处的值无意义，最终返回的是result->next
                    ListNode *result=new ListNode(0);
                    ListNode *newNode=result;
                    while(list1!=NULL && list2!=NULL){
                        if(list1->val < list2->val){
                            newNode->next=list1;
                            list1=list1->next;
                        }
                        else {
                            newNode->next=list2;
                            list2=list2->next;
                        }
                        newNode=newNode->next;
                    }
                    if(list1!=NULL){
                        newNode->next=list1;
                    }
                    else newNode->next=list2;
                    return result->next;
                }
            };
        ```

    - 堆排序：(算法无误，但是由于每次插入都需要重排，耗时很大，最终有些案列超时，所以还是使用二分合并为好)
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
            struct cmp{
                bool operator() (ListNode *l1,ListNode *l2){
                    return l1->val >= l2->val;
                }
            };
            ListNode* mergeKLists(vector<ListNode*>& lists) {
                if(lists.size()==0)
                    return NULL;
                //修改排序方式从小到大
                //priority_queue<Type, Container, Functional>
                priority_queue<ListNode, vector<ListNode*>, cmp> queue;

                //先把每个节点push进去
                for(int i=0;i<lists.size();i++){
                    ListNode *temp=lists[i];
                    while(temp!=NULL){
                        queue.push(temp);
                        temp=temp->next;
                    }
                }
                ListNode *result=NULL;   
                ListNode *temp=result;
                ListNode *newNode=NULL;
                while(!queue.empty()){
                    newNode=queue.top();
                    if(temp!=NULL)
                        temp->next=newNode;
                    else result=newNode;
                    temp=newNode;
                    queue.pop(); 
                }
                return result;
            }
        };
        ```
