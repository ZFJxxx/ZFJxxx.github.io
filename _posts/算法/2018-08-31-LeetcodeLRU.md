---
layout: post
title:  LeetCode LRU Cache
date:   2018-08-31 11:15:10
categories: 算法
tags: LeetCode
keywords: LeetCode
description: 
---

## 146. LRU Cache

**LRU是一种应用在操作系统上的缓存替换策略，和我们常见的FIFO算法一样，都是用于操作系统中内存管理中的页面替换，其全称叫做Least Recently Used（近期最少使用算法），算法主要是根据数据的历史访问记录来进行数据的淘汰，其核心思想是“如果数据最近被访问过，那么将来被访问的几率也更高”。**

```
Example:

LRUCache cache = new LRUCache( 2 /* capacity */ );  //容量为2

cache.put(1, 1);
cache.put(2, 2);
cache.get(1);       // returns 1
cache.put(3, 3);    // evicts key 2             //代替掉key 2
cache.get(2);       // returns -1 (not found)   //Key为2的已经被删除
cache.put(4, 4);    // evicts key 1             //代替掉key 1
cache.get(1);       // returns -1 (not found)
cache.get(3);       // returns 3
cache.get(4);       // returns 4
```

思路
```
添加(put)、查找(get)，因有容量限制，还需有删除，每次当容量满时，将最久未使用的节点删除。

为快速添加和删除，我们可以用双向链表来设计cache，链表中从头到尾的数据顺序依次是，(最近访问)->...(最旧访问)：

1）添加节点：新节点插入到表头即可，时间复杂度O(1)；

2）查找节点：每次节点被查询到时，将节点移动到链表头部，时间复杂度O(n)

可见在查找节点时，因对链表需遍历，时间复杂度O(n)，为达到O(1)，可以考虑数据结构中加入哈希(hash)。

=>我们需要用两种数据结构来解题：双向链表、哈希表
```
```
class Node {
    int key;
    int value;
    Node pre;
    Node next;

    public Node(int key, int value) {
        this.key = key;
        this.value = value;
    }
}

class LRUCache {
    HashMap<Integer, Node> map;
    int capicity, count;
    Node head, tail;
    public LRUCache(int capacity) {
        this.capicity = capacity;
	    map = new HashMap<>();
	    head = new Node(0, 0);
	    tail = new Node(0, 0);
	    head.next = tail;
	    tail.pre = head;
	    head.pre = null;
	    tail.next = null;
	    count = 0;
    }
    
    public int get(int key) {
        if (map.containsKey(key)) {
            Node node = map.get(key);
            deleteNode(node);             //删除该节点并移到头部
            addToHead(node);
            return node.value;
	    }
	    return -1;
    }
    
    public void put(int key, int value) {
        if (map.containsKey(key)) {       //如果该节点已经存在
            Node node = map.get(key);
            node.value = value;
            deleteNode(node);             //删除节点移到头部
            addToHead(node);
	    } else {
            Node node = new Node(key, value);
            map.put(key, node);
        }
        
		if (count < capicity) {
			count++;
			addToHead(node);
		} else {                          //如果超出Cache容量
			map.remove(tail.pre.key); 
			deleteNode(tail.pre);
			addToHead(node);
		}
	}
    
    public void deleteNode(Node node) {
        node.pre.next = node.next;
        node.next.pre = node.pre;
    }

    public void addToHead(Node node) {
        node.next = head.next;
        node.next.pre = node;
        node.pre = head;
        head.next = node;
    }
}

```
