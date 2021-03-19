---
title: "反转链表"
date: 2021-03-16T19:22:06+08:00
author: Xiak
image : "images/blog/web-browse.jpg"
bg_image: "images/featue-bg.jpg"
categories: ["Algorithm"]
tags: ["Leetcode", "List"]
description: ""
draft: false
type: "post"
---

### Leetcode 链接

[206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

### 题干
反转一个单链表。

示例 1：

```
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
```

> 进阶：你可以迭代或递归地反转链表。你能否用两种方法解决这道题？

### 思路

第一种方法: 通过自下而上的 `迭代` 实现

迭代比较好理解, 从第一个 `Node` 开始逐个反转 `Node` 指针, 时间复杂度为 `O(N)`, 空间复杂度为 `O(1)`

![reverse-list](/images/algorithm/reverse-list.png)


第二种方法: 通过自上而下的 `递归` 实现

递归比较绕, 计算机程序是通过栈实现递归, 人脑压栈出栈, 思考不了几层就容易晕掉, 所以不要去思考递归细节

写递归的关键就是分析递归函数的意义, 找出递推公式以及递归终止条件

- 递归函数意义: 输入链表头节点指针 `head`, 返回反转后的头节点指针 `newHead`
```golang
func reverse(head *ListNode) *ListNode {
    ......
    return newHead
}
```

- 递推公式: 链表反转, 可以想象为一个节点 `head` 和它之后的链表的头节点 `head.Next`, 尾节点 `newHead` 之间的关系
```golang
1. head -> head.Next -> newHead -> nil
2. head -> reverse(head.Next -> newHead -> nil)
3. head -> head.Next <- newHead
               |
               v
              nil
```
- 递归终止条件: `head.Next == nil`

整个递归过程如图所示

![reverse-list](/images/algorithm/reverse-list-recursion.png)


### 实现

迭代
```
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reverseList(head *ListNode) *ListNode {
    var pre, cur *ListNode = nil, head
    for cur != nil {
        head = head.Next
        cur.Next = pre
        pre = cur
        cur = head
    }
    return pre
}
```

递归
```golang
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reverseList(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }
    newHead := reverseList(head.Next)
    head.Next.Next = head
    head.Next = nil
    return newHead
}
```

### Benchmark 
```
迭代: 执行用时: 4 ms, 内存消耗: 2.5 MB
递归: 执行用时: 4 ms, 内存消耗: 3 MB
```
时间复杂度都为 `O(N)`

迭代为原地算法, 空间复杂度为 `O(1)`, 递归需要申请栈空间, 空间复杂度为 `O(N)`, 所以递归更占内存

如果考虑效率和成本, 反转链表算法用迭代更好.

### Leetcode 提交记录

[迭代](https://leetcode-cn.com/submissions/detail/155533737/)

[递归](https://leetcode-cn.com/submissions/detail/156025979/)

