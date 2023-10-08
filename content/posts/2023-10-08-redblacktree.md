---
title: "Golang 红黑树 redblacktree"
date: 2023-10-08T10:41:26+08:00
tags:
  - go package 
  - data structure
categories:
draft: false

---

[//]: # (# Golang红黑树 redblacktree)

红黑树（Red-Black Tree）是一种自平衡的二叉搜索树，它保持了在插入和删除操作时的平衡性，以确保树的高度保持在对数级别，从而保证了基本的查找、插入和删除操作的平均时间复杂度为O(log n)，其中n是树中的节点数量。红黑树在计算机科学中广泛用于实现高效的数据结构和算法，如集合、映射和动态有序集合。  
<!--more-->
在Leetcode中遇到了几次需要维护动态有序集合的场景，自实现的列表+二分查找增删的时间复杂度均为O(n)，经常导致超时，来学习下红黑树在go中的实现与使用。

## 红黑树原理
红黑树是一种自平衡的二叉搜索树，用于高效地实现插入、删除和查找操作。每个节点都带有一个额外的属性，表示其颜色，可以是红色或黑色。

红黑树必须满足以下关键性质：
- 根节点是黑色的。   
- 每个叶子节点（NIL节点）都是黑色的。   
- 如果一个节点是红色的，那么它的子节点必须是黑色的。   
- 不能有两个相邻的红色节点（即，红色节点不能连续出现）。   
- 从任意节点到其每个叶子节点的路径都包含相同数量的黑色节点，确保了树的平衡性。   

这些性质保证了红黑树的高度保持在可接受范围内，红黑树在插入和删除操作中通过旋转和染色保持这些关键性质，避免了退化成链表的情况。因此，红黑树在数据库、操作系统、编程语言编译器等领域得到广泛应用，为了快速和高效地管理数据而非常有用。其时间复杂度为O(log n)，使其成为一种强大的数据结构。

## redblacktree的使用
⚠️ redblacktree的红黑树的实现不是线程安全的。   
⚠️ redblacktree的红黑树只支持唯一key，即每个key只能对应唯一node，实现时需要注意。
### 红黑树的创建，插入和替换
redblacktree使用```NewWith(comparator utils.Comparator)```创建红黑树，支持传入用户自定义的comparator，也提供了```NewWithIntComparator()```和```NewWithStringComparator()```key为int和string类型的红黑树创建函数。
使用```Put(key interface{}, value interface{})```插入红黑树节点。key需要满足comparator的类型断言，否则会发生panic。
```go
func insert() {
    tree := rbt.NewWithIntComparator() // empty(keys are of type int)
    
    // Put 往红黑树中插入节点
    // key 需要满足comparator支持的类型，此处是int类型，如果不满足则panic
    tree.Put(1, "x") // 1->x
    tree.Put(2, "b") // 1->x, 2->b (in order)
    // 如果key已经存在则替换value
    tree.Put(1, "a") // 1->a, 2->b (in order, replacement)
    tree.Put(3, "c") // 1->a, 2->b, 3->c (in order)
    tree.Put(4, "d") // 1->a, 2->b, 3->c, 4->d (in order)
    tree.Put(5, "e") // 1->a, 2->b, 3->c, 4->d, 5->e (in order)
    tree.Put(6, "f") // 1->a, 2->b, 3->c, 4->d, 5->e, 6->f (in order)
    
    // RedBlackTree
    // │           ┌── 6
    // │       ┌── 5
    // │   ┌── 4
    // │   │   └── 3
    // └── 2
    //     └── 1
    fmt.Println(tree)
}
```
### 红黑树删除节点
redblacktree通过```Remove(key interface{})```删除节点。   
对不存在的key进行删除，不会产生副作用。
```go
{   ...    
    // RedBlackTree
    // │       ┌── 6
    // │   ┌── 5
    // └── 4
    //     │   ┌── 3
    //     └── 2
    tree.Remove(1)
    fmt.Println(tree)
}
```
### 获取红黑树中的最大最小值
```Left() *Node```获取最小值。   
```Right() *Node```获取最大值。
```go
{   
	... 
	// 2 
	minVal := tree.Left()
	// 6
	maxVal := tree.Right()
}
```
### 红黑树维护重复key 
由于redblacktree不支持重复key，所以需要支持重复key时，可以将value置为频数。实现如下：
```go
import (
	"fmt"

	rbt "github.com/emirpasic/gods/trees/redblacktree"
)

func main() {
	tree := rbt.NewWithIntComparator() // empty(keys are of type int)

	insert := func(key int, cnt int) {
		if v, ok := tree.Get(key); ok {
			nxt := v.(int) + cnt
			if nxt == 0 {
				tree.Remove(key)
			} else {
				tree.Put(key, nxt)
			}
		} else {
			tree.Put(key, cnt)
		}
	}

	// 插入
	insert(1, 1)
	insert(2, 1)
	insert(3, 1)
	insert(1, 1)
	
	// RedBlackTree
	// │   ┌── 3
	// └── 2
	//     └── 1
	fmt.Println(tree)

	// 删除
	insert(2, -1)
	
	// RedBlackTree
	// │   ┌── 3
	// └── 1
	fmt.Println(tree)
}
```

## 红黑树在工业中的应用
红黑树作为一种自平衡的二叉搜索树，在计算机科学和工程中有广泛的应用。以下是一些红黑树的常见应用：
>- Epoll 是 Linux 内核实现 IO 多路复用 (IO multiplexing) 的一个实现，是原先 poll/select 的改进版。Linux 中 epoll 的实现选择使用红黑树来储存文件描述符。
>- JDK 中的 TreeMap 和 TreeSet 都是使用红黑树作为底层数据结构的。同时在 JDK 1.8 之后 HashMap 内部哈希表中每个表项的链表长度超过 8 时也会自动转变为红黑树以提升查找效率。



## 参考
redblacktree https://pkg.go.dev/github.com/emirpasic/gods/trees/redblacktree#Tree.Put   
红黑树 https://oi-wiki.org/ds/rbtree/   
wiki https://zh.wikipedia.org/zh-hans/%E7%BA%A2%E9%BB%91%E6%A0%91
