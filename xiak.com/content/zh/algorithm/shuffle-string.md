---
title: "重新排列字符串"
date: 2021-03-12T19:17:00+08:00
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

[1528. 重新排列字符串](https://leetcode-cn.com/problems/shuffle-string/)

### 题干
给你一个字符串 s 和一个 长度相同 的整数数组 indices 。

请你重新排列字符串 s ，其中第 i 个字符需要移动到 indices[i] 指示的位置。

返回重新排列后的字符串。

示例 1：

![shuffle-string](/images/algorithm/shuffle-string.jpg)

```
输入：s = "codeleet", indices = [4,5,6,7,0,2,1,3]
输出："leetcode"
解释：如图所示，"codeleet" 重新排列后变为 "leetcode" 。
```

示例 2：
```
输入：s = "abc", indices = [0,1,2]
输出："abc"
解释：重新排列后，每个字符都还留在原来的位置上。
```

示例 3：
```
输入：s = "aiohn", indices = [3,1,4,2,0]
输出："nihao"
```

示例 4：
```
输入：s = "aaiougrt", indices = [4,0,2,6,7,3,1,5]
输出："arigatou"
```

示例 5：
```
输入：s = "art", indices = [1,0,2]
输出："rat"
```
> 提示：
> - s.length == indices.length == n
> - 1 <= n <= 100 
> - s 仅包含小写英文字母。
> - 0 <= indices[i] < n
> - indices 的所有的值都是唯一的（也就是说，indices 是整数 0 到 n - 1 形成的一组排列）

### 思路

算法
1. 常规算法： 新申请一个存储空间 (indices 大小)，遍历 `indices` , 把每个字符重新写入到新的内存空间中
2. 常规算法优化: 因为涉及到类型转换: `[]byte` 转 `string`, 其中需要内存的拷贝，可以使用 `*(*string)(unsafe.Pointer(&b))` 来实现零拷贝转换

### 实现

常规算法
```
func restoreString(s string, indices []int) string {
    l := len(indices)
    n := make([]byte, l)
    for i, v := range indices{
        n[v] = s[i]
    }
    return string(n[0:l])
}
```

常规算法丶改
```golang
func SliceByteToString(b []byte) string {
	return *(*string)(unsafe.Pointer(&b))
}

func restoreString(s string, indices []int) string {
    l := len(indices)
    n := make([]byte, l)
    for i, v := range indices{
        n[v] = s[i]
    }
    return SliceByteToString(n[0:l])
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

