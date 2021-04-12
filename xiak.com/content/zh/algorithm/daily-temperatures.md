---
title: "739. 每日温度"
date: 2021-04-12T13:55:00+08:00
author: Xiak
image : "images/blog/web-browse.jpg"
bg_image: "images/featue-bg.jpg"
categories: ["Algorithm"]
tags: ["Leetcode", "KMP"]
description: ""
draft: false
type: "post"
---

### Leetcode 链接

[739. 每日温度]

### 题干
请根据每日 `气温` 列表，重新生成一个列表。对应位置的输出为：要想观测到更高的气温，至少需要`等待的天数`。如果气温在这之后都不会升高，请在该位置用 `0` 来代替。

例如，
```
给定一个列表 temperatures = [73, 74, 75, 71, 69, 72, 76, 73]

你的输出应该是 [1, 1, 4, 2, 1, 1, 0, 0]
```


> 提示：气温 列表长度的范围是`[1, 30000]`。每个气温的值的均为华氏度，都是在`[30, 100]`范围内的整数。

### 思路

#### 暴力迭代原地算法
输入是一个列表，输出也是一个列表，且长度相同，考虑使用原地算法

遍历列表 temperatures, 每个数字和它所在位置之后的所有数据比较，找到比它大的数字，他们之间的步长即为结果，存入当前数字位置 (不用额外分配内存)，如果整个列表没有比它大的数字，则返回 0

- 最坏时间复杂度为 O(xy), x 为列表长度, y 为 x 后的所有数据长度, 
- 空间复杂度 O(1)

#### KMP 算法

[如何更好地理解和掌握 KMP 算法?](https://www.zhihu.com/question/21923021)

[Bilibili KMP算法易懂版](https://www.bilibili.com/video/BV1jb411V78H?from=search&seid=9213718721515748230)

假设数组长度为 `length := len(temperatures)`

设置两个指针 `w (walk): 从列表倒数第二位开始，向列表头遍历` 和 `c (current): 目的是找到 walk 指针后最大的数字`

还需要一个数组 `next: 存放每个位置气温升高需要等待的天数`

例如:
```
Index: 0   1   2   3   4   5   6   7
List:  73  74  75  71  69  75  76  73

初始化 next 数组
next:  0   0   0   0   0   0   0   0

第一次比较
Index: 0   1   2   3   4   5   6   7
List:  73  74  75  71  69  75  76  73
                               w   c   76 > 73, 查看 next[7] 为 0 
                                       表示知道 73 之后没有比 73 更大的数字了, 
                                       而 76 > 73, 所以 next[6] = 0
next:                          0   0
                               
第二次比较
Index: 0   1   2   3   4   5   6   7
List:  73  74  75  71  69  75  76  73
                           w   c       75 < 76, 找到比 75 更大的数 76, 
                                       步长为 next[5] = c - w = 1
next:                      1    0   0 

第三次比较
Index: 0   1   2   3   4   5   6   7
List:  73  74  75  71  69  75  76  73
                       w   c           69 < 75, 找到比 69 更大的数 75, 
                                       步长为 next[4] = c - w = 1
next:                  1   1   0   0     

第四次比较
Index: 0   1   2   3   4   5   6   7
List:  73  74  75  71  69  75  76  73
                   w   c               71 > 69, 需要找到比 71 大的数字, 先找比 69 大的数的位置
                   w       c           next[4] = 1 表示比 69 大的数字位置为 c=4+1=5, 值为 75
                                       71 < 75, 找到比 71 大的数了, 步长为 next[3] = c - w = 2
next:              2   1   1   0   0     

第五次比较
Index: 0   1   2   3   4   5   6   7
List:  73  74  75  71  69  75  76  73
               w   c                   75 > 71, 需要找到比 75 大的数字, 先找比 71 大的数的位置
               w           c           next[3] = 2 表示比 71 大的数字位置为 c=3+2=5, 值为 75
                                       75 == 75, 继续找比 75(List[5]) 大的数
                                       next[5] = 1, c=c+next[5]=5+1=6, 值为 76
               w               c       75 < 76, 找到比 75 大的数了, 步长为 next[2] = c - w = 4
next:          4   2   1   1   0   0     
               
第六次比较
Index: 0   1   2   3   4   5   6   7
List:  73  74  75  71  69  75  76  73
           w   c                       74 < 75, 找到比 74 更大的数 75,
                                       步长为 next[1] = c - w = 2 - 1 = 1
next:      1   4   2   1   1   0   0


第七次比较
Index: 0   1   2   3   4   5   6   7
List:  73  74  75  71  69  75  76  73
       w   c                           73 < 74, 找到比 73 更大的数 74, 
                                       步长为 next[0] = c - w = 1 - 0 = 1
next:  1   1   4   2   1   1   0   0  
   
比较完成 next 数组即为所求
```

- 时间复杂度为 O(x+y)
- 空间复杂度为 O(x)

#### 单调栈

(暂略)

### 实现

```golang
package algorithm

// 方法一: 暴力迭代
// 时间复杂度 O(MN), 空间复杂度 O(1)
// 运行时间 1108 ms, 内存消耗 7 MB
func dailyTemperatures(T []int) []int {
	length := len(T)
	for i := 0; i < length - 1; i++ {
		for j := i + 1; j < length; j++ {
			if T[i] < T[j] {
				T[i] = j - i
				break
			}
			if j == length - 1 {
				T[i] = 0
				break
			}
		}
	}
	T[length-1] = 0
	return T
}

// 方法二: 类 KMP
// 时间复杂度 O(m+n). 空间复杂度 O(n)
// 运行时间 60 ms, 内存消耗 7.4 MB
func dailyTemperatures(T []int) []int {
	length := len(T)
	next := make([]int, length)
	var w, c int
	// walk and current 
	for w = length - 2; w >= 0; w-- {
		c = w + 1
		// 后面的数比当前数小, 根据 next 数组找到比 T[c] 更大的数再做比较
		for T[w] >= T[c] {
			if next[c] != 0 {
				c += next[c]
            } else {
            	break
            }
        }
		if T[w] < T[c] {
			next[w] = c - w
        }
    }
    return next
}
```

### Leetcode 提交记录

[常规算法](https://leetcode-cn.com/submissions/detail/166887360/)
[KMP](https://leetcode-cn.com/submissions/detail/166893930/)


