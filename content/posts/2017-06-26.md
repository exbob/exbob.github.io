---
title: LeetCode 刷题
date: 2017-06-26T08:00:00+08:00
draft: false
toc:
comments: true
---


> 用 [LeetCode](https://leetcode.com) 练习 Python ，难度都是选择 Easy 。


## 1. Two Sum

Given an array of integers, return indices of the two numbers such that they add up to a specific target.

You may assume that each input would have exactly one solution, and you may not use the same element twice.

Example:

    Given nums = [2, 7, 11, 15], target = 9,
    
    Because nums[0] + nums[1] = 2 + 7 = 9,
    return [0, 1].
    
Answer：

    class Solution(object):
    def twoSum(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """
        for i in range(len(nums)):
            for j in range(i+1,len(nums)):
                if nums[i]+nums[j]==target:
                    return [i,j]

## 2. Reverse Integer

Reverse digits of an integer.

Example1: x = 123, return 321
Example2: x = -123, return -321

Note:The input is assumed to be a 32-bit signed integer. Your function should return 0 when the reversed integer overflows.
    
Answer:

    class Solution(object):
        def reverse(self, x):
            """
            :type x: int
            :rtype: int
            """
            temp=abs(x)
            res=0
            while temp>0:
                res=res*10+temp%10
                temp=temp//10
            if x<0:
                res=-res
            if res>2147483647 or res<-2147483648 :
                res=0
            return res
            
## 3. Palindrome Number

Determine whether an integer is a palindrome. Do this without extra space.

判断一个整数是否是回文数，尽可能的少用额外的内存空间。一个思路是将数字折半，对比两半是否相等。

Answer:

    class Solution(object):
        def isPalindrome(self, x):
            """
            :type x: int
            :rtype: bool
            """
            if x<0 :
                return False
            if x//10==0 :
                return True
            if x%10==0 :
                return False
            
            s=0
            while x>s :
                s=s*10+x%10
                x=x//10
    
            if x==s or x==s//10 :
                return True
            else :
                return False
                
还有一种方法是转换为字符串，然后从头尾开始对比字符:

    class Solution(object):
        def isPalindrome(self, x):
            """
            :type x: int
            :rtype: bool
            """
            x=str(x)
            l=len(x)
            for i in range(l):
                if x[i] != x[-1-i]:
                    return False
            return True
            
还可以在数组中利用 Python 的冒号语法，将数字前后颠倒，然后判断是否相同：

    class Solution(object):
        def isPalindrome(self, x):
            """
            :type x: int
            :rtype: bool
            """
            x = str(x)
            if x == x[::-1]:
                return True
            else:
                return False            

## 4. Roman to Integer

Given a roman numeral, convert it to an integer.

Input is guaranteed to be within the range from 1 to 3999.

Answer:

    class Solution(object):
        def romanToInt(self, s):
            """
            :type s: str
            :rtype: int
            """
            rtoi={'I':1,'V':5,'X':10,'L':50,'C':100,'D':500,'M':1000}
            buf=0
            for i in range(len(s)-1):
                if rtoi[s[i]] < rtoi[s[i+1]]:
                    buf=buf-rtoi[s[i]]
                else:
                    buf=buf+rtoi[s[i]]
            buf=buf+rtoi[s[-1]]
            return buf
            
## 5. Longest Common Prefix

Write a function to find the longest common prefix string amongst an array of strings.

找出字符串数组中的最长公共前缀，比如 array=['abc','abcd','abe']，最长公共前缀就是 'ab' 。这道题的思路是先找出数组中最短的字符串，长度为 max ，最长公共前缀不可能超过 max ，然后遍历 max 次字符串数组，依次对比前 max 个字符。

Answer:

    class Solution(object):
        def longestCommonPrefix(self, strs):
            """
            :type strs: List[str]
            :rtype: str
            """
            res=''
            strs_len=len(strs)
            if strs_len<1:
                return res
            if strs_len==1:
                return strs[0]
            
            max=len(strs[0])
            for i in range(strs_len):
                if len(strs[i])<max:
                    max=len(strs[i])
            if max<1:
                return res
                
            flag=0
            for i in range(max):
                for j in range(1,strs_len):
                    print i,j
                    print strs[j][i]
                    if strs[0][i]!=strs[j][i]:
                        flag=1
                        break
                if flag==0:
                   res+=strs[0][i]
                else:
                   break
            return res

## 6. Valid Parentheses

Given a string containing just the characters '(', ')', '{', '}', '[' and ']', determine if the input string is valid.

The brackets must close in the correct order, "()" and "()[]{}" are all valid but "(]" and "([)]" are not.

验证括号是否完全匹配。思路是遍历整个字符串，遇到左括号就压栈，遇到右括号就与栈内括号对比，空栈或者不匹配就说明这个字符串无效。

Answer:

    class Solution(object):
        def isValid(self, s):
            """
            :type s: str
            :rtype: bool
            """
            if len(s)<2 or len(s)%2!=0 :
                return False
    
            a=[['{','[','('],['}',']',')']]
            b={'}':'{',']':'[',')':'('}
            
            t=[]
            for i in range(len(s)):
                if s[i] in a[0] :
                    t.append(s[i])
                else:
                    if len(t)==0 :
                        return False
                    l=t.pop()
                    if l!=b[s[i]] :
                        return False
            if len(t)==0 :
                return True
            else :
                return False
                
## 7. Remove Duplicates from Sorted Array
    
Given a sorted array, remove the duplicates in place such that each element appear only once and return the new length.

Do not allocate extra space for another array, you must do this in place with constant memory.

For example,
Given input array nums = [1,1,2],

Your function should return length = 2, with the first two elements of nums being 1 and 2 respectively. It doesn't matter what you leave beyond the new length.

Answer:

    class Solution(object):
        def removeDuplicates(self, nums):
            """
            :type nums: List[int]
            :rtype: int
            """
            i=0
            while True :
                if i<len(nums)-1 :
                    if nums[i]==nums[i+1] :
                        nums.pop(i+1)
                    else :
                        i+=1
                else :
                    break
            
            return len(nums)

## 8. Remove Element

Given an array and a value, remove all instances of that value in place and return the new length.

Do not allocate extra space for another array, you must do this in place with constant memory.

The order of elements can be changed. It doesn't matter what you leave beyond the new length.

Example:

Given input array nums = [3,2,2,3], val = 3

Your function should return length = 2, with the first two elements of nums being 2.

Answer:

    class Solution(object):
        def removeElement(self, nums, val):
            """
            :type nums: List[int]
            :type val: int
            :rtype: int
            """
            i=0
            while i<len(nums):
                if nums[i]==val :
                    nums.pop(i)
                else:
                    i+=1
            return len(nums)

## 9. Implement strStr()

Returns the index of the first occurrence of needle in haystack, or -1 if needle is not part of haystack.

Answer:

    class Solution(object):
        def strStr(self, haystack, needle):
            """
            :type haystack: str
            :type needle: str
            :rtype: int
            """
            needle_len=len(needle)
            haystack_len=len(haystack)
            if needle_len==0 :
                return 0
            i=0
            while i+needle_len<=haystack_len :
                if needle==haystack[i:i+needle_len] :
                    return i
                else :
                    i+=1
            return -1
            
其实 Python 为字符串提供了一个 `find()` 方法:

     string.find(str,beg=0,end=len(string))
     
用于查找字符串 string 内是否包含另一个字符串 str ，如果是返回开始的索引值，否则返回 -1 。

## 10. Search Insert Position

Given a sorted array and a target value, return the index if the target is found. If not, return the index where it would be if it were inserted in order.

You may assume no duplicates in the array.

Here are few examples.

    [1,3,5,6], 5 → 2
    [1,3,5,6], 2 → 1
    [1,3,5,6], 7 → 4
    [1,3,5,6], 0 → 0

Answer:

    class Solution(object):
        def searchInsert(self, nums, target):
            """
            :type nums: List[int]
            :type target: int
            :rtype: int
            """
            for i in range(len(nums)) :
                if target<=nums[i] :
                    return i
            return len(nums)
            
## 11. Count and Say

The count-and-say sequence is the sequence of integers with the first five terms as following:

    1.     1
    2.     11
    3.     21
    4.     1211
    5.     111221
    
`1` is read off as `one 1` or `11`.

`11` is read off as `two 1s` or `21`.

`21` is read off as `one 2, then one 1` or `1211`.

Given an integer n, generate the nth term of the count-and-say sequence.

Note: Each term of the sequence of integers will be represented as a string.

Example 1:

    Input: 1
    Output: "1"
    
Example 2:

    Input: 4
    Output: "1211"
    
Answer:

    class Solution(object):
        def countAndSay(self, n):
            """
            :type n: int
            :rtype: str
            """
            s='1'
            for i in range(n-1) :
                pre=s[0]
                t=''
                count=0
                for a in s :
                    if pre==a :
                        count+=1
                    else :
                        t+=str(count)
                        t+=pre
                        count=1
                        pre=a
            
                t+=str(count)
                t+=pre
                s=t
            return s    
