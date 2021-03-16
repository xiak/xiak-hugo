---
title: "重新排列字符串"
date: 2021-03-15T19:20:00+08:00
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

[https://leetcode-cn.com/problems/reverse-linked-list/](https://leetcode-cn.com/problems/reverse-linked-list/)

### 题干
反转一个单链表。

示例 1：

```
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
```

> 进阶：你可以迭代或递归地反转链表。你能否用两种方法解决这道题？

### 思路

思路如图所示

![reverse-list](/images/algorithm/reverse-list.png)

算法
1. 迭代
2. 递归

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

### Benchmark 
```
go test  -benchmem -test.bench=".*"
goos: windows
goarch: amd64
BenchmarkShuffleString_1-4      24535662                46.9 ns/op            16 B/op          2 allocs/op
BenchmarkShuffleString_2-4      43719510                27.5 ns/op             8 B/op          1 allocs/op
```

可以看到 常规算法丶改各方面性能都是最优的, 而且少了一次内存分配
```golang
// src/runtime/string.go
// The constant is known to the compiler.
// There is no fundamental theory behind this number.
const tmpStringBufSize = 32
type tmpBuf [tmpStringBufSize]byte
func slicebytetostring(buf *tmpBuf, b []byte) (str string) {
    ......
    if buf != nil && len(b) <= len(buf) {
        p = unsafe.Pointer(buf)
    } else {
        p = mallocgc(uintptr(len(b)), nil, false)
    }
    ......
    memmove(p, (*(*slice)(unsafe.Pointer(&b))).array, uintptr(len(b)))
    ......
}
```
从上面 `golang` 的源代码可以看出, 通过 `string([]byte)` 把 `[]byte` 强制转换为 `string` 时候, 只要 `[]byte` 长度超过 32，就会通过 `mallocgc` 申请新的内存, 然后通过 `memmove` 做一次内存搬运

而通过 `*(*string)(unsafe.Pointer(&b))` 作类型转换时，其本质只是改变了指针指向而已，没有做内存申请和搬运，所以性能要好很多

### Leetcode 提交记录

[https://leetcode-cn.com/submissions/detail/154381214/](https://leetcode-cn.com/submissions/detail/154381214/)

