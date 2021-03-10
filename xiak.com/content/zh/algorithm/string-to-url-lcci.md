---
title: "字符串 URL 化"
date: 2021-03-10T18:39:00+08:00
author: Xiak
image : "images/blog/light.jpg"
bg_image: "images/featue-bg.jpg"
categories: ["Algorithm"]
tags: ["Leetcode","Interview"]
description: ""
draft: false
type: "post"
---

此题目为 [leetcode](https://leetcode-cn.com) 上的面试题

### Leetcode 链接

[https://leetcode-cn.com/problems/string-to-url-lcci/](https://leetcode-cn.com/problems/string-to-url-lcci/)

### 题干
URL化。编写一种方法，将字符串中的空格全部替换为%20。假定该字符串尾部有足够的空间存放新增字符，并且知道字符串的“真实”长度。（注：用Java实现的话，请使用字符数组实现，以便直接在数组上操作。）

示例 1：
```
输入："Mr John Smith    ", 13
输出："Mr%20John%20Smith"
```

示例 2：
```
输入："               ", 5
输出："%20%20%20%20%20"
```

> 提示：
  字符串长度在 [0, 500000] 范围内。

### 思路

#### 数据结构
- 分析 golang 底层字符串结构
- 在字符串的基础上替换空格

#### 算法


### 实现

```
func replaceSpaces(S string, length int) string {
    
}
```

### Leetcode 提交记录

[https://leetcode-cn.com/submissions/detail/83261636/](https://leetcode-cn.com/submissions/detail/83261636/)

