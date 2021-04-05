---
title: "链表设计"
date: 2021-03-19T15:30:00+08:00
author: Xiak
image : "images/blog/cafe.jpg"
bg_image: "images/featue-bg.jpg"
categories: ["Algorithm"]
tags: ["Leetcode", "List"]
description: ""
draft: false
type: "post"
---

### Leetcode 链接

[707. 设计链表](https://leetcode-cn.com/problems/design-linked-list/)

### 题干
设计链表的实现。您可以选择使用单链表或双链表。单链表中的节点应该具有两个属性：val 和 next。val 是当前节点的值，next 是指向下一个节点的指针/引用。如果要使用双向链表，则还需要一个属性 prev 以指示链表中的上一个节点。假设链表中的所有节点都是 0-index 的。

在链表类中实现这些功能：

- get(index)：获取链表中第 index 个节点的值。如果索引无效，则返回-1。
- addAtHead(val)：在链表的第一个元素之前添加一个值为 val 的节点。插入后，新节点将成为链表的第一个节点。
- addAtTail(val)：将值为 val 的节点追加到链表的最后一个元素。
- addAtIndex(index,val)：在链表中的第 index 个节点之前添加值为 val  的节点。如果 index 等于链表的长度，则该节点将附加到链表的末尾。如果 index 大于链表长度，则不会插入节点。如果index小于0，则在头部插入节点。
- deleteAtIndex(index)：如果索引 index 有效，则删除链表中的第 index 个节点。

示例:
```golang
MyLinkedList linkedList = new MyLinkedList();
linkedList.addAtHead(1);
linkedList.addAtTail(3);
linkedList.addAtIndex(1,2);   //链表变为1-> 2-> 3
linkedList.get(1);            //返回2
linkedList.deleteAtIndex(1);  //现在链表是1-> 3
linkedList.get(1);            //返回3
```

提示:

> - 所有 val 值都在 [1, 1000] 之内
> - 操作次数将在 [1, 1000] 之内
> - 请不要使用内置的 LinkedList 库


### 思路

链表是底层的数据结构, 本文将会以清晰易懂的方式带领大家实现一个双向链表(每个节点都可以访问之前的节点和之后的节点), 我会把文章分为两部分

- 数据结构设计

- 链表方法设计

#### 数据结构设计

题目给出了一下结构体
```golang
type MyLinkedList struct {
}
```

带头尾的双向链表则可以表示为 
```golang
type MyLinkedList struct {
    head *MyLinkedList
    tail *MyLinkedList
    pre  *MyLinkedList
    next *MyLinkedList
    val  int
}
```

可是这样设计好吗? 链表每个节点都必须存一个 `head` 和 `tail` 吗? 答案是否定的, 它们只需要存一次就够了, 那我们把上面结构体拆一下
```golang
type ListNode struct {
    pre  *ListNode
    next *ListNode
    val  int
}

type MyLinkedList struct {
    head   *ListNode
    tail   *ListNode
    length int
}
```

把一个 `node` 必须的 `pre: 前置指针`, `next: 后置指针` 和 `val: 值` 独立为一个结构体 `ListNode`, 表示 `link list` 的一个节点

而 `MyLinkedList` 则专注存放链表属性 `head: 链表头节点`, `tail: 链表尾节点` 和 `length: 链表的长度`

#### 链表方法设计

- `Constructor()`: 初始化链表

初始化带头尾的双向链表, 需要分别初始化头, 尾节点, 再把它们连接起来

![list-constructor](/images/algorithm/list-constructor.png)

代码实现

```golang
func Constructor() MyLinkedList {
    // 初始化头
    head := &ListNode { nil, nil, 0 }
    // 初始化尾
    tail := &ListNode { head, nil, 0 }
    // 头尾连接
    head.next = tail
    return MyLinkedList { 
        head: head, 
        tail: tail, 
        length: 0,
    }
}
```

- `AddAtHead(val)`: 在链表的第一个元素之前添加一个值为 val 的节点。插入后，新节点将成为链表的第一个节点。

在链表第一个元素之前插入节点, 也就是说在 `head` 节点值后, 在 `第一个节点` 之前插入节点

时间复杂度为 `O(1)`, 空间复杂度为 `O(1)`

![list-add-node-at-head](/images/algorithm/list-add-node-at-head.png)

代码实现
```golang
func (this *MyLinedList) AddAtHead (val int) {
    // 新节点, pre = this.head, next = this.head.next
    new := &ListNode{ 
        pre: this.head, 
        next: this.head.next, 
        val: val,
    }
    // 更新 head 和 第一个节点 的指针, 指向 new 节点
    new.next.pre = new
    this.head.next = new
    // 链表长度加　1
    this.length ++
}
```


- `AddAtTail(val)`: 将值为 val 的节点追加到链表的最后一个元素。

在链表最后一个元素之后插入节点, 即在 `tail` 之前, 在 `最后一个元素` 之后插入节点

时间复杂度为 `O(1)`, 空间复杂度为 `O(1)`

![list-add-node-at-tail](/images/algorithm/list-add-node-at-tail.png)

代码实现
```golang
func (this *MyLinkedList) AddAtTail(val int) {
    // 新节点
    new := &ListNode{ 
        pre: this.tail.pre, 
        next: this.tail, 
        val: val, 
    }
    // 更新 tail 和 最后一个节点的指针
    new.pre.next = new
    this.tail.pre = new
    // 链表长度加　1
    this.length ++
}

```

- `AddAtIndex(index,val)`: 在链表中的第 index 个节点之前添加值为 val  的节点。如果 index 等于链表的长度，则该节点将附加到链表的末尾。如果 index 大于链表长度，则不会插入节点。如果index小于0，则在头部插入节点。

在链表指定位置插入一个节点, 这种情况需要遍历链表

时间复杂度为 `O(n)`, 空间复杂度为 `O(1)`

![list-add-node-at-index](/images/algorithm/list-add-node-at-index.png)

代码实现
```golang
func (this *MyLinedList) AddAtIndex(index, val int) {
    // 如果 index 长度大于链表长度 this.length, 不做操作
    if index > this.length {
        return
    }
    // 如果 index 长度等于链表长度, 在链表末尾添加此元素
    if index == this.length {
        this.AddAtTail(val)
        return
    }
    // 遍历链表, 让 cur 指针指向第 index 个节点
    cur := this.head.next
    for i := 0; i < index; i ++ {
        cur = cur.next
    }
    // 在第 index 个节点前插入新节点
    new := &ListNode{ 
        pre: cur.pre, 
        next: cur, 
        val: val,
    }
    new.pre.next = new
    new.next.pre = new
    // 链表长度加　1
    this.length ++
}
```

- `DeleteAtIndex(index)`: 如果索引 index 有效，则删除链表中的第 index 个节点。

在指定位置删除一个节点, 这种情况同样需要遍历

时间复杂度为 `O(n)`, 空间复杂度为 `O(1)`

![list-delete-node-at-index](/images/algorithm/list-delete-node-at-index.png)

```golang
func (this *MyLinkedList) DeleteAtIndex(index int) {
    // 注意这里是 >= (比如 100 个节点, index 范围为 0-99, 当 index == 100 时已经越界)
    if index >= this.length {
        return
    }
    // 把 cur 指针指向第 index 个元素
    cur := this.head.next
    for i := 0; i < index; i ++ {
        cur = cur.next
    }
    cur.next.pre = cur.pre
    cur.pre.next = cur.next
    cur.pre = nil
    cur.next = nil
    this.length --
}
```

- `Get(index)`: 获取链表中第 index 个节点的值。如果索引无效，则返回-1。

获取指定位置的节点信息, 需要遍历链表, 找到位置返回节点值即可

时间复杂度为 `O(n)`, 空间复杂度为 `O(1)`

```golang
func (this *MyLinkedList) Get (index int) int {
    if index >= this.length {
        return -1
    }
    cur := this.head.next
    for i := 0; i < index; i ++ {
        cur = cur.next
    }
    return cur.val
}

```


### 实现

```golang
package list

import "fmt"

type ListNode struct {
	val  int
	pre  *ListNode
	next *ListNode
}

type MyLinkedList struct {
	head   *ListNode
	tail   *ListNode
	length int
}


/** Initialize your data structure here. */
func Constructor() MyLinkedList {
	// 初始化头
    head := &ListNode{ 0, nil, nil }	
    // 初始化尾
    tail := &ListNode{ 0, head, nil }
    // 头尾连接
    head.next = tail
    return MyLinkedList { 
        head: head, 
        tail: tail, 
        length: 0,
    }
}


/** Get the value of the index-th node in the linked list. If the index is invalid, return -1. */
func (this *MyLinkedList) Get(index int) int {
    if index >= this.length {
        return -1
    }
    cur := this.head.next
    for i := 0; i < index; i ++ {
        cur = cur.next
    }
    return cur.val
}


/** Add a node of value val before the first element of the linked list. After the insertion, the new node will be the first node of the linked list. */
func (this *MyLinkedList) AddAtHead(val int)  {
    // 新节点, pre = this.head, next = this.head.next
    new := &ListNode{ 
        pre: this.head,
        next: this.head.next,
        val: val,
    }
    // 更新 head 和 第一个节点 的指针, 指向 new 节点
    new.next.pre = new
    this.head.next = new
    // 链表长度加　1
    this.length ++
}


/** Append a node of value val to the last element of the linked list. */
func (this *MyLinkedList) AddAtTail(val int)  {
    // 新节点
    new := &ListNode{ 
        pre: this.tail.pre,
        next: this.tail,
        val: val,
    }
    // 更新 tail 和 最后一个节点的指针
    new.pre.next = new
    this.tail.pre = new
    // 链表长度加　1
    this.length ++
}


/** Add a node of value val before the index-th node in the linked list. If index equals to the length of linked list, the node will be appended to the end of linked list. If index is greater than the length, the node will not be inserted. */
func (this *MyLinkedList) AddAtIndex(index int, val int)  {
    // 如果 index 长度大于链表长度 this.length, 不做操作
    if index > this.length {
        return
    }
    // 如果 index 长度等于链表长度, 在链表末尾添加此元素
    if index == this.length {
        this.AddAtTail(val)
        return
    }
    // 遍历链表, 让 cur 指针指向第 index 个节点
    cur := this.head.next
    for i := 0; i < index; i ++ {
        cur = cur.next
    }
    // 在第 index 个节点前插入新节点
    new := &ListNode{ 
        pre: cur.pre,
        next: cur,
        val: val,
    }
    new.pre.next = new
    new.next.pre = new
    // 链表长度加　1
    this.length ++
}


/** Delete the index-th node in the linked list, if the index is valid. */
func (this *MyLinkedList) DeleteAtIndex(index int)  {
    // 注意这里是 >= (比如 100 个节点, index 范围为 0-99, 当 index == 100 时已经越界)
    if index >= this.length {
        return
    }
    // 把 cur 指针指向第 index 个元素
    cur := this.head.next
    for i := 0; i < index; i ++ {
        cur = cur.next
    }
    cur.next.pre = cur.pre
    cur.pre.next = cur.next
    cur.pre = nil
    cur.next = nil
    this.length --
}

func (this *MyLinkedList) Show() {
	cur := this.head.next
	for i := 0; i < this.length; i++ {
		fmt.Printf("%d ", cur.val)
		cur = cur.next
	}
	fmt.Printf("\n")
}

/**
 * Your MyLinkedList object will be instantiated and called as such:
 * obj := Constructor();
 * param_1 := obj.Get(index);
 * obj.AddAtHead(val);
 * obj.AddAtTail(val);
 * obj.AddAtIndex(index,val);
 * obj.DeleteAtIndex(index);
 */
```

### Benchmark 
```
执行用时: 28 ms
内存消耗: 7 MB
```

### Leetcode 提交记录

[记录](https://leetcode-cn.com/submissions/detail/157177919/)


