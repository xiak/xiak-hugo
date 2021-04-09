---
title: "28. 实现 strStr"
date: 2021-04-09T18:30:00+08:00
author: Xiak
image : "images/blog/book.jpg"
bg_image: "images/featue-bg.jpg"
categories: ["Algorithm"]
tags: ["Leetcode"]
description: ""
draft: false
type: "post"
---

### Leetcode 链接

[28. 实现 strStr](https://leetcode-cn.com/problems/implement-strstr/)

### 题干
实现 `strStr()` 函数。

给定一个 `haystack` 字符串和一个 `needle` 字符串，在 `haystack` 字符串中找出 `needle` 字符串出现的第一个位置 (从0开始)。如果不存在，则返回 -1。


示例 1:

```
输入: haystack = "hello", needle = "ll"
输出: 2
```

示例 2:

```
输入: haystack = "aaaaa", needle = "bba"
输出: -1
```


> 说明:
> - 当 `needle` 是空字符串时，我们应当返回什么值呢？这是一个在面试中很好的问题。
> - 对于本题而言，当 needle 是空字符串时我们应当返回 0 。这与C语言的 strstr() 以及 Java的 indexOf() 定义相符。

### 思路

分析: 字符串匹配的问题有几种解法

- BF (略)

- BM (待续)

- KMP

[如何更好地理解和掌握 KMP 算法?](https://www.zhihu.com/question/21923021)

[Bilibili KMP算法易懂版](https://www.bilibili.com/video/BV1jb411V78H?from=search&seid=9213718721515748230)

KMP 比较难理解, 其关键就是找 `next` 数组, 可以参考以上文章了解

### 实现

```go
package kpm

func GetNext(p string) []int {
	length := len(p)
    next := make([]int, length)
    next[0] = -1
    i, j := 0, -1
    for i < length - 1 {
        if j == -1 || p[i] == p[j] {
        	i++
        	j++
        	next[i] = j
        } else {
        	j = next[j]
        }   
    }
    return next
}

func Kpm(t string, p string) int {
	pLen := len(p)
	if pLen == 0 {
		return 0
    }
    tLen := len(t)
    i, j := 0, 0
    next := GetNext(p)
    if i < tLen && j < pLen {
    	if j == -1 || t[i] == p[j] {
    		i++
    		j++
        } else {
        	j = next[j]
        }
    }
    
    if j == pLen {
    	return i - j
    } else {
    	return -1
    }
}
```

### Benchmark 
```
执行用时：0 ms, 在所有 Go 提交中击败了 100.00% 的用户
内存消耗：2.5 MB, 在所有 Go 提交中击败了 13.27% 的用户
```

### Leetcode 提交记录

[常规算法](https://leetcode-cn.com/submissions/detail/165794544/)


