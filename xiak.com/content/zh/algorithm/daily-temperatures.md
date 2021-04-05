---
title: "739. 每日温度"
date: 2021-03-22T16:30:00+08:00
author: Xiak
image : "images/blog/work.jpg"
bg_image: "images/featue-bg.jpg"
categories: ["Algorithm"]
tags: ["Leetcode"]
description: ""
draft: false
type: "post"
---

### Leetcode 链接

[739. 每日温度](https://leetcode-cn.com/problems/daily-temperatures/)

### 题干
请根据每日 `气温` 列表，重新生成一个列表。对应位置的输出为：要想观测到更高的气温，至少需要`等待的天数`。如果气温在这之后都不会升高，请在该位置用 `0` 来代替。

例如，
```
给定一个列表 temperatures = [73, 74, 75, 71, 69, 72, 76, 73]

你的输出应该是 [1, 1, 4, 2, 1, 1, 0, 0]
```


> 提示：气温 列表长度的范围是`[1, 30000]`。每个气温的值的均为华氏度，都是在`[30, 100]`范围内的整数。

### 思路

#### 暴力算法

#### 单调栈

### 实现

非原地算法
```golang

```

### Benchmark 
```
原地算法:
执行用时：8 ms, 在所有 Go 提交中击败了 92.82% 的用户
内存消耗：4.4 MB, 在所有 Go 提交中击败了 99.94%的用户

非原地算法:
执行用时：8 ms, 在所有 Go 提交中击败了 92.82% 的用户
内存消耗：4.6 MB, 在所有 Go 提交中击败了 99.44% 的用户
```
时间复杂度都为 `O(Max(L1, L2))`

原地算法, 没有额外的空间产生, 空间复杂度为 `O(1)`

非原地算法, 分配了一个额外存储链表, 长度为 `MAX(L1, L2)` 空间复杂度为 `O(MAX(L1, L2))`

> 原地算法使用前提: 如果实际开发中需要保持输入的两个链表 `L1 和 L2` 的值不变的话, 不能用原地算法

### Leetcode 提交记录

[常规算法](https://leetcode-cn.com/submissions/detail/158329734/)


