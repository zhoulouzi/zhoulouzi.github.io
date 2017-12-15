---
title: "leetcode 笔记"
date: 2017-09-05T21:48:15+08:00
draft: false
slug: "leetcode"
tags:
- leetcode
archives:
- leetcode
categories:
- algorithm
clearReading: true
thumbnailImage: https://res.cloudinary.com/ddvxfzzbe/image/upload/v1513355321/Real_gaggav.png
thumbnailImagePosition: right
autoThumbnailImage: yes
metaAlignment: center
comments: true
showTags: true
showPagination: true
showSocial: true
showDate: true
---


# 总结出一些比较意义的题
> 好久没有更新自己的博客了，自从3月份跳槽到现在这几个月一直很忙，所以也一直没有时间更新，最近自己也抽空去leetcode刷题，补一补薄弱的环节。就从easy难度的开始刷起，刷完这600多道题。

1. 一个列表成员都是出现2次的整数，只有一个元素出现一次。
> Given an array of integers, every element appears twice except for one. Find that single one.
> Note:
> Your algorithm should have a linear runtime complexity. Could you implement it without using extra memory?
我的解法很平常遍历一遍列表给每个元素计数，返回值为1的元素，但是并不符合Note里提到的。于是在大神们的Solutions找到了这个答案：
> One-line python solution with O(n) time
```
def singleNumber(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        return reduce(lambda x, y: x ^ y, nums)
```
第一眼看，什么鬼，x 的 y次方～ 想了半天才明白 相同2个数 异或运算结果就是0 0和任意数 异或运算都是 任意数本身啊。  reduce下这个列表，完美。
2. Remove Element
Given an array and a value, remove all instances of that value in place and return the new length.
Do not allocate extra space for another array, you must do this in place with constant memory.
The order of elements can be changed. It doesn't matter what you leave beyond the new length.
Example:
Given input array nums = [3,2,2,3], val = 3
Your function should return length = 2, with the first two elements of nums being 2.
>  这个题用python有点简单。但是 list.remove这个方法用的还是很少的.加深下记忆
```
def removeElement(nums, val):
     for i in range(nums.count(val)):
            nums.remove(val)
     return len(nums)
```
3. Palindrome Number 回数，又看见这个题了，曾经面试遇到过
利用切片搞定。
```
def isPalindrome(x):
        """
        :type x: int
        :rtype: bool
        """
        return str(x) == str(x)[::-1]
```
4. Valid Palindrome 回文，忽略大小写，只考虑字母和数字
> For example,
"A man, a plan, a canal: Panama" is a palindrome.
"race a car" is not a palindrome.
```
 def isPalindrome(s):
        """
        :type s: str
        :rtype: bool
        """
        act = []
        act = list(filter(lambda a: a.isalnum(), list(s.lower())))
        return act == act[::-1]
```


排序：
冒泡排序：
```
arr = [9,8,7,6,5,4,3,2,1]
for j in range(len(arr)):
    for i in range(len(arr) - 1 - j ):
        if arr[i] > arr[i+1]:
            arr[i],arr[i+1] = arr[i+1],arr[i]

print(arr)
```

快速排序：
```
def quick(L,left,right):
    if left >= right:
        return L
    key = L[left]
    low = left
    high = right
    while left < right:
        while left < right and L[right] >= key:
            right -= 1
        L[left] = L[right]                     #------------------ 比key值小的值放到左边
        while left <right and L[left] <= key:
            left += 1
        L[right] = L[left]                     #------------------比key值大的放到右边
    #left = right 跳出循环
    L[left] = key                             #------------------将key值赋给left=right的位置
    #此时left=right,分别对左右子列表进行快速排序
    quick(L,low,right-1)
    quick(L,left+1,high)
    #递归调用，知道子列表为单一子列表 left=right
quick(L,0,len(L)-1)
```



