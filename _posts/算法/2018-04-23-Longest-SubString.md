---
layout: post
title:  LeetCode 3. Longest Substring Without Repeating Characters
date:   2018-04-22 11:15:10
categories: 算法
tags: LeetCode
keywords: LeetCode
description: 
---

Given a string, find the length of the longest substring without repeating characters.

Examples:

Given "abcabcbb", the answer is "abc", which the length is 3.

Given "bbbbb", the answer is "b", with the length of 1.

Given "pwwkew", the answer is "wke", with the length of 3. Note that the answer must be a substring, "pwke" is a subsequence and not a substring.

## 解

"滑动窗口" 
比方说 abcabccc 当你右边扫描到abca的时候你得把第一个a删掉得到bca，
然后"窗口"继续向右滑动，每当加到一个新char的时候，左边检查有无重复的char，
然后如果没有重复的就正常添加，
有重复的话就左边扔掉一部分（从最左到重复char这段扔掉），在这个过程max中记录最大窗口长度

最精髓的是 leftBound = Math.max(leftBound,map.get(c)+1); 这一行代码
这样就不用每次更新新的子串的时候都去刷新HashMap,而是将leftBound移到最新的重复节点处。
```
class Solution {
    public int lengthOfLongestSubstring(String s) {
        if (s.length()==0) return 0;
        //新建一个map进行存储char
        HashMap<Character, Integer> map = new HashMap<Character, Integer>();
        int max=0;
        int leftBound=0;
        for (int i=0; i<s.length(); ++i){
            Character c = s.charAt(i);
            //窗口左边可能为下一个char，或者不变
            if (map.containsKey(s.charAt(i))){
                leftBound = Math.max(leftBound,map.get(c)+1);
            }
            map.put(c ,i);
            max = Math.max(max,i-leftBound+1);  //窗口长度最大值
        }
        return max;
    }
}
```
