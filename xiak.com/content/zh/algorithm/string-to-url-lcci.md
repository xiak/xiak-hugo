---
title: "字符串 URL 化"
date: 2021-03-11T19:50:00+08:00
author: Xiak
image : "images/blog/light.jpg"
bg_image: "images/featue-bg.jpg"
categories: ["Algorithm"]
tags: ["Leetcode","Interview"]
description: ""
draft: false
type: "post"
---

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

数据结构

golang 中的 `string` 本质为`固定长度`的`只读`的切片

```golang
type StringHeader struct {
  Data uintptr
  Len  int
}
```

![golang-string](/images/algorithm/golang-string.png)

要修改字符串必须先将 `string` 转换为 `[]byte` 或者 `[]rune`

算法

1. 直接用类库 `strings.Replace` 或则 `bytes.Buffer`
2. 常规算法： 新申请一个存储空间 (3倍 length 大小)，遍历字符串 `length` 长度, 把每个字符重新写入到新的内存空间中
3. 常规算法优化: 因为涉及到类型转换: `[]byte` 转 `string`, 其中需要内存的拷贝，可以使用 `*(*string)(unsafe.Pointer(&b))` 来实现零拷贝转换

### 实现

这里总结了 `4` 种实现方法

使用 `golang` 库 `strings.Replace`
```
func ReplaceSpaces_1(S string, length int) string {
	return strings.Replace(S[:length], " ", "%20", -1)
}
```

使用 `golang` 库 `bytes.Buffer`
```
func ReplaceSpaces_2(S string, length int) string {
	var buffer bytes.Buffer
	for i := 0; i < length; i++ {
		if S[i] == ' ' {
			buffer.WriteString("%20")
		} else {
			buffer.WriteByte(S[i])
		}
	}
	return buffer.String()
}
```
常规算法
```golang
func ReplaceSpaces_3(S string, length int) string {
	t := make([]byte, length*3)
	w := 0
	for i := 0; i < length; i++ {
		if S[i] == ' ' {
			copy(t[w:(w+3)], "%20")
			w += 3
		} else {
			t[w] = S[i]
			w++
		}
	}
	return string(t[0:w])
}
```
常规算法丶改
```golang
func SliceByteToString(b []byte) string {
	return *(*string)(unsafe.Pointer(&b))
}

func ReplaceSpaces_4(S string, length int) string {
	t := make([]byte, length*3)
	w := 0
	for i := 0; i < length; i++ {
		if S[i] == ' ' {
			copy(t[w:(w+3)], "%20")
			w += 3
		} else {
			t[w] = S[i]
			w++
		}
	}
	return SliceByteToString(t[0:w])
}
```

### Benchmark 
```
go test  -benchmem -test.bench=".*"
goos: windows
goarch: amd64
BenchmarkReplaceSpaces_1-4      10319277               115 ns/op              64 B/op          2 allocs/op
BenchmarkReplaceSpaces_2-4       9039098               130 ns/op              96 B/op          2 allocs/op
BenchmarkReplaceSpaces_3-4      15924559                72.8 ns/op            80 B/op          2 allocs/op
BenchmarkReplaceSpaces_4-4      25308819                46.3 ns/op            48 B/op          1 allocs/op
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

[https://leetcode-cn.com/submissions/detail/153905677/](https://leetcode-cn.com/submissions/detail/153905677/)

