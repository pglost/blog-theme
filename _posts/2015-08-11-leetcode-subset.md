---
layout: post
title:  "LeetCode:排列组合类型问题总结"
date:   2015-08-10 13:30:54
categories: leetcode
banner-url: "/public/images/post-banner/algorithm.jpg"
---

* content
{:toc}

##写在最前

最近在刷leetcode的过程中，发现排列组合类型的问题，都可以使用回朔递归的方法来解决。下面对刷题过程中遇到的排列组合类问题加以总结和整理。


##基本问题:子集 [Subsets](https://leetcode.com/problems/subsets/)

###问题描述

Given a set of distinct integers, nums, return all possible subsets.

给定一个含不同整数的集合，返回其所有的子集

### 问题分析

这是这一类问题中的基础问题，其他排列组合的问题都可以套用本题的解题思路来解决。

###代码

	class Solution {
	public:
	    vector<vector<int>> subsets(vector<int>& nums) {
	        sort(nums.begin(),nums.end());
	        vector<vector<int>> subs;
	        vector<int> sub;
	        subsetsHelper(nums,0,sub,subs);
	        return subs;
	    }
	private:
	    void subsetsHelper(vector<int>& nums,int pos,vector<int> sub,vector<vector<int>>& subs){
	        subs.push_back(sub);
	        for(int i = pos;i < nums.size();i++){
	            sub.push_back(nums[i]);
	            subsetsHelper(nums,i+1,sub,subs);
	            sub.pop_back();
	        }
	    }

	};

###代码要点
1. 因为题目中要求子集中的元素为非降序排列，因此要首先对题目给出的集合进行排序。
2. subsetsHelper(nums,i,sub,subs)的含义是，从nums集合的第i个元素开始，计算包含sub的所有子集，并把所有的子集保存到subs中。
3. 要想解决subsetsHelper(nums,i,sub,subs)这个问题，就必须解决它的子问题，那就是对于nums中从i开始的每一个元素，先把它保存到sub中，然后调用subsetsHelper(nums,i+1,sub,subs)。最后把这个思路转换成上述代码。

##扩展1：带重复元素的子集 [Subsets II](https://leetcode.com/problems/subsets-ii/)

###问题描述
Given a collection of integers that might contain duplicates, nums, return all possible subsets.

给定一个可能具有重复数字的列表，返回其所有可能的子集

###问题分析


这个问题与[Subsets](https://leetcode.com/problems/subsets/)类似，只要把重复的的子集排除就可以。例如：{1, 2(1), 2(2), 2(3)},规定{1, 2(1)}和{1, 2(2)}重复,{1, 2(1), 2(2)}和{1, 2(2), 2(3)}重复。从而得出规律：:我们只关心取多少个2,不关心取哪几个。因此，规定必须从第一个2开始连续取(作为重复集合中的代表),如必须是{1, 2(1)}不能是{1, 2{2})。

###代码

	class Solution {
	public:
	    vector<vector<int>> subsetsWithDup(vector<int>& nums) {
	     sort(nums.begin(), nums.end());
	     vector<vector<int>> subs;
	     vector<int> sub;
	     subsetsWithDupHelper(nums, 0, sub, subs);
	     return subs;
	    }
	private:
	   void subsetsWithDupHelper(vector<int>& nums, int pos, vector<int>& sub, vector<vector<int>>& subs) {
	        subs.push_back(sub);
	        for(int i = pos; i < nums.size(); i++){
	            if(i != pos && nums[i] == nums[i-1]){
	                continue;
	            }
	            sub.push_back(nums[i]);
	            subsetsWithDupHelper(nums, i+1, sub, subs);
	            sub.pop_back();
	        }

	    }
	};

###代码要点
1. 还是首先要对集合排序，首先是因为题目要求输出的子集为非升序的，其次排序之后很容易的就把重复的子集排除。
2. subsetsWithDupHelper(nums, i, sub, subs)的含义是，从nums的第i个元素开始，计算包含sub的所有不重复子集，并把结果存储到subs中。
3. 如何保证计算的子集不重复？在subsetsWithDupHelper(nums, i, sub, subs)函数中，在每一层的递归中，把与前面元素重复的元素跳过。

##扩展2：全排列 [Permutations](https://leetcode.com/problems/permutations/)

Given a collection of numbers, return all possible permutations.

给定一个数字列表，返回其所有可能的排列。

###问题描述

与subsets的思路类似，可以再subsets模板的基础上加以修改。

###代码

	class Solution {
	public:
	    vector<vector<int>> permute(vector<int>& nums) {
	        vector<vector<int>> result;
	        vector<int> list;
	        permuteHelper(nums, list, result);
	        return result;
	    }
	private:
	    void permuteHelper(vector<int>& nums, vector<int>& list, vector<vector<int>>& result){
	        if(list.size() == nums.size()){
	            result.push_back(list);
	            return;
	        }
	        for(int i = 0; i < nums.size(); i++){
	            if(find(list.begin(),list.end(),nums[i]) != list.end()){
	                continue;
	            }
	            list.push_back(nums[i]);
	            permuteHelper(nums, list, result);
	            list.pop_back();
	        }
	    }
	};

###代码要点
在subsets的基础上，做两点修改

1. 每次递归当且仅当sub的长度与nums的长度相同时，才把sub存入到subs中。
2. 每次递归需要把nums中所有元素遍历一遍，只有当nums中的元素在sub中包含时才跳过（排除在上上层已经参与递归的元素）

##扩展3：带重复元素的排列 [Permutations II](https://leetcode.com/problems/permutations-ii/)

###问题描述

Given a collection of numbers that might contain duplicates, return all possible unique permutations.

给出一个具有重复数字的列表，找出列表所有不同的排列

###问题分析

在Permutations的基础上，把重复的组合排除，并且对已经在前一层参与递归nums的元素做排除，由于nums中有重复的元素，因此不能使用Permutations中方法，可以使用一个bool数组标识nums元素的递归状态。

###代码

	class Solution {
	public:
	    vector<vector<int>> permuteUnique(vector<int>& nums) {
	        vector<vector<int>> result;
	        vector<int> list;
	        vector<bool> visited(nums.size(), false);
	        sort(nums.begin(),nums.end());
	        permuteUniqueHelper(nums, list, result,visited);
	        return result;
	    }
	private:
	    void permuteUniqueHelper(vector<int>& nums, vector<int>& list, vector<vector<int>>& result ,vector<bool>& visited){
	        if(list.size() == nums.size()){
	            result.push_back(list);
	            return;
	        }
	        for(int i = 0; i < nums.size(); i++){
	            if(visited[i] || (i != 0 && nums[i] == nums[i-1] && !visited[i-1])){
	                continue;
	            }
	            visited[i] = true;
	            list.push_back(nums[i]);
	            permuteUniqueHelper(nums, list ,result,visited);
	            list.pop_back();
	            visited[i] = false;
	        }
	    }
	};

###代码要点

1. 排除上层已经递归的元素
2. 排除本层中重复出现的元素

##扩展4：组合数字 [Combination Sum](https://leetcode.com/problems/combination-sum/)

###问题描述

Given a set of candidate numbers (C) and a target number (T), find all unique combinations in C where the candidate numbers sums to T.

The same repeated number may be chosen from C unlimited number of times.

给出一组候选数字(C)和目标数字(T),找到C中所有的组合，使找出的数字和为T。

C中的数字可以无限制重复被选取。

###问题分析

subsets II的变形，需要考虑给出的候选集合存在重复元素的情况。递归返回的条件以及需要跳过的情况与subsets II略有不同

###代码

	class Solution {
	public:
	    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
	        vector<vector<int>> result;
	        vector<int> list;
	        sort(candidates.begin(), candidates.end());
	        combinationSumHelper(candidates, target, 0,list, result);
	        return result;
	    }
	private:
	    void combinationSumHelper(vector<int>& candidates, int target, int start,vector<int>& list,vector<vector<int>>& result){
	        if (target == 0) {
	            result.push_back(list);
	            return;
	        }
	        for (int i = start; i < candidates.size(); i++) {
	            if (candidates[i] > target){
	                break;
	            }
	            if (i !=start && candidates[i-1] == candidates[i]) {
	                continue;
	            }
	            list.push_back(candidates[i]);
	            combinationSumHelper(candidates, target - candidates[i], i, list, result);
	            list.pop_back();
	        }
	    }
	};

###代码要点
1. combinationSumHelper(candidates, target, i,list, result)的含义：从candidates的第i个元素开始，选择包含list的并且满足子集的和为target的所有非重复子集，并且把找到的集合存到result中。
2. 递归返回的条件为target为0。

##扩展5：字符串组合问题 [Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)

###问题描述

Given a digit string, return all possible letter combinations that the number could represent.

A mapping of digit to letters (just like on the telephone buttons) is given below.

###问题分析

第一层递归，digits中第一个数字对应的字符集；第二层递归，包含第一层选中的字符，对digits的第二个数字做第二层递归。

###代码

	class Solution {
	public:
	    vector<string> letterCombinations(string digits) {
	        vector<string> result;
	        if(digits.size() == 0){
	            return result;
	        }
	        string list;
	        vector<string> map = {"", "" , "abc", "def", "ghi", "jkl", "mno", "pqrs","tuv","wxyz"};
	        letterCombinationsHelper(digits, map, 0,list, result);
	        return result;
	    }
	    void letterCombinationsHelper(string& digits, vector<string>& map,int index, string list, vector<string>& result) {
	        if (index == digits.size()) {
	            result.push_back(list);
	            return;
	        }
	        for(int i = 0; i < map[digits[index]-'0'].size(); i++) {
	            list.push_back(map[digits[index]-'0'][i]);
	            letterCombinationsHelper(digits, map, index+1,list, result);
	            list.pop_back();
	        }
	    }
	};

###代码要点

1. C++中vector的初始化不能用[]的形式
2. 各种C++中字符串的操作

##扩展6：字符串分割之一 [Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/)

###题目描述
Given a string s, partition s such that every substring of the partition is a palindrome.

Return all possible palindrome partitioning of s.

给定一个字符串s，将s分割成一些子串，使每个子串都是回文串。

返回s所有可能的回文串分割方案。

###问题分析

第一层递归为s的以首字母开头的所有连续子串，第二层递归为上一层递归后一个字符开的所有连续子串。如果子串不为回文，则停止子递归。递归返回的条件为走到s的最后一个字符。

###代码

	class Solution {
	public:
	    vector<vector<string>> partition(string s) {
	        vector<vector<string>> result;
	        vector<string> list;
	        partitionHelper(s, 0, list, result);
	        return result;
	    }
	private:
	    bool isPalindrome(const string& s) {
	        int begin = 0;
	        int end = s.size() - 1;
	        while (begin < end) {
	            if(s[begin] != s[end]) {
	                return false;
	            }
	            begin++;
	            end--;
	        }
	        return true;
	    }
	    void partitionHelper(string& s, int start, vector<string>& list, vector<vector<string>>& result) {
	        if(start == s.size()){
	            result.push_back(list);
	            return;
	        }
	        for(int i = start; i < s.size(); i++){
	            string prefix = s.substr(start,i - start + 1);
	            if(!isPalindrome(prefix)) {
	                continue;
	            }
	            list.push_back(prefix);
	            partitionHelper(s, i+1, list, result);
	            list.pop_back();
	        }
	    }
	};

###代码要点
1.回文字符串的判断

##扩展7：字符串分割之二 [Restore IP Addresses](https://leetcode.com/problems/restore-ip-addresses/)

### 问题描述

Given a string containing only digits, restore it by returning all possible valid IP address combinations.

###问题分析

基本同上一题类似，递归返回条件为分割了四个子串，如果递归到了s的末尾，就把当前分割存储到result中。

###代码

	class Solution {
	public:
	    vector<string> restoreIpAddresses(string s) {
	        vector<string> result;
	        vector<string>  list;
	        restoreIpAddressesHelper(s, 0, list, result);
	        return result;
	    }
	private:
	    bool isValid(string s) {
	        if (s[0] == '0') {
	            return s == "0";
	        }
	        int strValue = stoi(s);
	        return strValue >= 0 && strValue <= 255;
	    }
	    void restoreIpAddressesHelper(string& s, int start, vector<string>& list, vector<string>& result) {
	        if(list.size() == 4) {
	            if(start != s.size()) {
	                return;
	            }
	            string listCombine;
	            for(int i = 0; i < list.size(); i++){
	                listCombine = listCombine + list[i] + ".";
	            }
	            listCombine = listCombine.substr(0,listCombine.size()-1);
	            result.push_back(listCombine);
	            return;
	        }
	        for(int i = start; i < s.size() && i < start + 4; i++) {
	            string prefix = s.substr(start,i - start + 1);
	            if(isValid(prefix)){
	                list.push_back(prefix);
	                restoreIpAddressesHelper(s, i+1, list, result);
	                list.pop_back();
	            }

	        }
	    }

	};

###代码要点：

1.字符串的操作

##总结

￼排列组合模板的使用范围
1. 几乎所有的搜索问题 根据具体题目要求进行改动
2. 什么时候输出
3. 哪些情况需要跳过
4. 上一层的递归和下一层递归的关系








