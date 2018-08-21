---
layout: post
title:  Leetcode 链表问题
date:   2018-04-22 11:15:10
categories: 算法
tags: LeetCode
keywords: LeetCode
description: 
---
## 19. Remove Nth Node From End of List
**删除到数第k个节点**
```
Example:

Given linked list: 1->2->3->4->5, and n = 2.

After removing the second node from the end, the linked list becomes 1->2->3->5.
```
思路：
```
两个指针，相距n+1,当快指针到尾节点的时候，慢指针刚好到到数第n+1个节点。

注意删除第一个节点的情况，需要设置头节点指向head
```
```
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {

        ListNode start = new ListNode(0);
        ListNode slow = start, fast = start;
        slow.next = head;

        for(int i=0; i<=n; i++)   {
            fast = fast.next;
        }
        while(fast != null) {
            slow = slow.next;
            fast = fast.next;
        }

        slow.next = slow.next.next;
        return start.next;
    }
}
```
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

## 876. Middle of the Linked List
**找到链表的中间元素**
```
Example 1:

Input: [1,2,3,4,5]
Output: Node 3 from this list (Serialization: [3,4,5])
The returned node has value 3.  (The judge's serialization of this node is [3,4,5]).
Note that we returned a ListNode object ans, such that:
ans.val = 3, ans.next.val = 4, ans.next.next.val = 5, and ans.next.next.next = NULL.
Example 2:

Input: [1,2,3,4,5,6]
Output: Node 4 from this list (Serialization: [4,5,6])
Since the list has two middle nodes with values 3 and 4, we return the second one.
```
思路
```
用快慢指针，快的每次走俩格，慢的每次走一格。

快的走到结尾，慢的刚好走到中间。
```
```
class Solution {
    public ListNode middleNode(ListNode head) {
        ListNode slow =head;
        ListNode fast =head;
        while(fast!= null && fast.next!=null){
            slow = slow.next;
            fast = fast.next.next;
        }
        return slow;
    }
}
```
