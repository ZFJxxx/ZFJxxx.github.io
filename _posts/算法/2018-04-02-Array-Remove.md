---
layout: post
title:  Leetcode 链表问题
date:   2018-04-22 11:15:10
categories: 算法
tags: LeetCode
keywords: LeetCode
description: 
---

## 83. Remove Duplicates from Sorted List
**删除链表中重复的元素**
```
```
Example 1:

Input: 1->1->2
Output: 1->2
Example 2:

Input: 1->1->2->3->3
Output: 1->2->3
```
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if(head == null || head.next == null)
            return head;
        ListNode cur = head;
        while(cur.next != null){
            if(cur.val == cur.next.val){
                cur.next = cur.next.next;
            }else{
                cur = cur.next; 
            }
        }
        return head;
    }
}
```
