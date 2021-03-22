---
title: "两数相加: 常规算法和原地算法"
date: 2021-03-22T16:30:00+08:00
author: Xiak
image : "images/blog/work.jpg"
bg_image: "images/featue-bg.jpg"
categories: ["Algorithm"]
tags: ["Leetcode", "List"]
description: ""
draft: false
type: "post"
---

### Leetcode 链接

[2. 两数相加](https://leetcode-cn.com/problems/add-two-numbers/)

### 题干
给你两个 `非空` 的链表，表示两个非负的整数。它们每位数字都是按照 `逆序` 的方式存储的，并且每个节点只能存储 `一位` 数字。

请你将两个数相加，并以相同形式返回一个表示和的链表。

你可以假设除了数字 `0` 之外，这两个数都不会以 `0` 开头。


示例 1：

![add-two-numbers](/images/algorithm/add-two-num-1.jpg)

```
输入：l1 = [2,4,3], l2 = [5,6,4]
输出：[7,0,8]
解释：342 + 465 = 807.
```

示例 2：

```
输入：l1 = [0], l2 = [0]
输出：[0]
```

示例 3：

```
输入：l1 = [9,9,9,9,9,9,9], l2 = [9,9,9,9]
输出：[8,9,9,9,0,0,0,1]
```


> 提示：
> - 每个链表中的节点数在范围 [1, 100] 内
> - 0 <= Node.val <= 9
> - 题目数据保证列表表示的数字不含前导零

### 思路

分析: 

1. 链表每个节点存储的数字在 `0~9` 之间, 意味着当两数相加大于 `9` 后, 需要进位
2. 节点数量不大, 可以新开辟空间存储相加后的结果

有几种思路可以实现需求

第一种方法: 常规算法

动态生成一个新链表, 存储两数相加的结果

时间复杂度为 `O(MAX(L1, L2))`, 空间复杂度为 `O(MAX(L1, L2))`

第二种方法: 原地算法

不额外申请一个链表, 重用已经存在的两个链表, 当 2 数相加后, 同时更新两个链表的值

时间复杂度为 `O(MAX(L1, L2))`, 空间复杂度为 `O(1)`


### 实现

非原地算法
```golang
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
    // 第一个 node
    result := &ListNode{ 0, nil }
    cur := result
    var sum, carry int = 0, 0
    for l1 != nil || l2 != nil {
        sum = carry
        if l1 != nil {
            sum = sum + l1.Val
            l1 = l1.Next
        } 
        if l2 != nil {
            sum = sum + l2.Val
            l2 = l2.Next
        }
        if sum < 10 {
            carry = 0
        } else {
            carry = 1
            sum = sum - 10
        }
        cur.Val = sum
        if l1 == nil && l2 == nil {
            if carry == 1 {
                cur.Next = &ListNode{ 1, nil }
            }
            break
        }
        cur.Next = &ListNode{ 0, nil }
        cur = cur.Next
    }
    return result
}
```

原地算法
```
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
    // 用 pL1, pL2 遍历数组, 用 cur 指向最长的链表节点
    var pL1, pL2, pCur, pCurHead *ListNode = l1, l2, l1, l1
    var sum, carry int = 0, 0
    // 1: l1 != nil && l2 == nil
    // 2: l1 == nil && l2 != nil
    // 3: l1 != nil && l2 != nil
    // 0: l1 == nil && l2 == nil
    var code int
    for {
        sum = carry
        code = 0
        if pL1 != nil {
            sum += pL1.Val
            code = 1
        }
        if pL2 != nil {
            sum += pL2.Val
            code = code + 2
        }
        if sum < 10 {
            if pL1 != nil || pL2 != nil {
                carry = 0
            }
        } else {
            carry = 1
            sum = sum - 10
        }
        if code == 1 {
            pL1.Val = sum
            pCur, pCurHead = pL1, l1
            pL1 = pL1.Next
        } else if code == 2 {
            pL2.Val = sum
            pCur, pCurHead = pL2, l2
            pL2 = pL2.Next   
        } else if code == 3 {
            pL1.Val, pL2.Val = sum, sum
            pCur, pCurHead = pL1, l1
            pL1, pL2 = pL1.Next, pL2.Next
        } else {
            if carry == 1 {
                pCur.Next = &ListNode{
                    Val: 1,
                }
            }
            break
        }
    }
    return pCurHead
}
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

[原地算法](https://leetcode-cn.com/submissions/detail/158291521/)

