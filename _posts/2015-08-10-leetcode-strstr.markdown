---
layout: post
title:  "LeetCode: Implement strStr()"
date:   2015-08-10 13:30:54
categories: leetcode
banner-url: "/public/images/post-banner/algorithm.jpg"
---

* content
{:toc}

##题目

Returns the index of the first occurrence of needle in haystack, or -1 if needle is not part of haystack.

##问题分析

本题为字符串匹配问题。KMP等高级算法算法实现起来比较复杂，最简单实用的方法还是Brute Force。

##解题要点

1. 对haystack做第一层循环。
2. 对needle做第二层循环，通过循环判断needle和haystack是否匹配：如果有一个字符不匹配则跳出第二层循环，继续第一层循环；如果第二层循环走到了最后，则匹配成功。并返回此时haystack中字符的位置。
3. 如果第一层循环结束之后函数还没有返回值，则匹配失败，返回-1。

##代码
	class Solution {
	public:
	    int strStr(string haystack, string needle) {
	        int m = haystack.length();
	        int n = needle.length();

	        for(int i = 0;i < m - n + 1;i++){
	            int j = 0;
	            for(;j < n;j++){
	                if(needle[j] != haystack[i+j]){
	                    break;
	                }
	            }
	            if (j == n){
	                return i;
	            }
	        }
	        return -1;
	    }
	};
