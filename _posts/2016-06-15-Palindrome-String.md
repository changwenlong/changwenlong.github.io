---
layout: post
title:  "构造回文串"
date:   2016-06-15
author:  
categories: 算法编程
excerpt: 低版本IE的bug和兼容性，点击空块级元素时
---

* content
{:toc}

题目来源：[腾讯2017实习生招聘编程题](http://www.nowcoder.com/profile/592799/test/3401660/44802)

## 题目描述

给定一个字符串s，你可以从中删除一些字符，使得剩下的串是一个回文串。如何删除才能使得回文串最长呢？

输出需要删除的字符个数。

## 解题思路

比较简单的想法就是求原字符串和其反串的最大公共子串的长度，然后用原字符串的长度减去这个最大公共子串的长度就得到了最小编辑长度。（注：最大公共子串并不一定要连续的，只要保证出现次序一致即可看作公共子串）。

求两个字符串的最大公共子串可采用dp实现。

dp的状态转化方程为：

- dp[i][j]=dp[i-1][j-1]+1, str1[i]==str2[j]
- dp[i][j]=max(dp[i-1][j],dp[i][j-1]), str1[i]!=str2[j]

## 代码实现

    package edu.zju.chwl.tencent;
    
    import java.util.*;
    
    public class Main {
    
        public static void main(String[] args) {
            Scanner in = new Scanner(System.in);
            while(in.hasNext()){
                String str=in.next();
                int len = str.length();
                System.out.println(len-getLongestCommonLen(str,len));;
            }
    
        }
    
        private static int getLongestCommonLen(String str1,int n) {
            String str2 = getReverseStr(str1,n);
            int[][] dp = new int[n+1][n+1];
            if(str1.charAt(0)==str2.charAt(0)){
                dp[1][1]=1;
            }
            for(int i=1;i<=n;i++){
                for(int j=1;j<=n;j++){
                    if(str1.charAt(i-1)==str2.charAt(j-1)){
                        dp[i][j]=dp[i-1][j-1]+1;
                    }else{
                        dp[i][j]=Math.max(dp[i-1][j], dp[i][j-1]);
                    }
                }
            }
            return dp[n][n];
        }
    
        private static String getReverseStr(String str1, int n) {
            StringBuilder sb = new StringBuilder();
            for(int i=n-1;i>=0;i--){
                sb.append(str1.charAt(i));
            }
            return sb.toString();
        }
    }

