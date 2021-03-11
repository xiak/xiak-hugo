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

![golang-string](images/algorithm/golang-string.png)

要访问字符串中某个或者某段字符可以通过以下方式:
```golang
x := "Mr Xiak"
fmt.Println(x[0], x[3]) // "77 88" ('M' and 'X')
fmt.Println(x[:2]) // "Mr"
fmt.Println(x[3:]) // "Xiak"
fmt.Println(x[:])  // "Mr Xiak"
fmt.Println("hi" + x[2:]) // "Hi Xiak"
```
要修改字符串必须先将 `string` 转换为 `[]byte` 或者 `[]rune`

算法



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

第 4 种是第三种方法的变种
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

### Leetcode 提交记录

[https://leetcode-cn.com/submissions/detail/153905677/](https://leetcode-cn.com/submissions/detail/153905677/)

