---
layout: post
title:  LeetCode 718.Longest Common Substring(Medium)
date:   2018-07-10 22:15:10
categories: 算法
tags: LeetCode
keywords: 
description: 
---

找两个字符串的最长公共子串，这个子串要求在原字符串中是连续的。

## 解

其实这是一个序贯决策问题，可以用动态规划来求解。使用一个二维矩阵来记录中间结果。 
eg： abcd和bca

![](http://p7lixluhf.bkt.clouddn.com/LCS.PNG)

```
public class Solution {
     /*
     * @param A, B: Two string.
     * @return: the length of the longest common substring.
     */
    public int longestCommonSubstring(String A, String B) {
        if(A==null || B==null) return 0;
        int max=0;                      //记录最长子串的大小
        char[] AStr=A.toCharArray();
        char[] BStr=B.toCharArray();
        int lenA=AStr.length;
        int lenB=BStr.length;
      
        int dp[][]=new int[lenA][lenB];  //初始化矩阵
        for(int i=0;i<lenA;i++){
            for(int j=0;j<lenB;j++){
                if(AStr[i]==BStr[j]){  //当charA = charB时
                    if(i==0||j==0){
                        dp[i][j]=1;
                    }else{
                        dp[i][j]=a[i-1][j-1]+1;
                    }
                    max = Math.max(max,dp[i][j]);
                }
                else dp[i][j]=0;
            }
        }
        return max;
    }
}
```
