---
title: "Golang sort.Search二次学习"
date: 2023-10-27T11:01:37+08:00
tags:
  - go package
categories:
draft: false

---

[//]: # (# Golang sort.Search二次学习)
做题时，使用golang的sort.Search进行二分查找时，返回的结果令人疑惑，来重新学习下sort.Search的用法。
<!--more-->

## 令人疑惑的返回
运行以下代码，发现使用sort.Search查找切片[2,4,5]中的第一个等于2的元素返回index为3，3表示数组中没有符合```func(i int) bool { return arr[i] == 2 }```的元素，结果与事实不相符。在数组中查找等于4的元素却得到了合理的答案，得到返回为1。一个算法正确性不应当依赖于输入，让我们一起看看sort.Search究竟做了些什么。
```go
func main() {
    arr := []int{2, 4, 5}
    index := sort.Search(len(arr), func(i int) bool { return arr[i] == 1 })
    fmt.Println(index) //3
    index = sort.Search(len(arr), func(i int) bool { return arr[i] <= 1 })
    fmt.Println(index) //3
    index = sort.Search(len(arr), func(i int) bool { return arr[i] >= 1 })
    fmt.Println(index) //0
    
    index = sort.Search(len(arr), func(i int) bool { return arr[i] == 2 })
    fmt.Println(index) //3
    index = sort.Search(len(arr), func(i int) bool { return arr[i] <= 2 })
    fmt.Println(index) //3
    index = sort.Search(len(arr), func(i int) bool { return arr[i] >= 2 })
    fmt.Println(index) //0
    
    index = sort.Search(len(arr), func(i int) bool { return arr[i] == 4 })
    fmt.Println(index) //1
    index = sort.Search(len(arr), func(i int) bool { return arr[i] <= 4 })
    fmt.Println(index) //0
    index = sort.Search(len(arr), func(i int) bool { return arr[i] >= 4 })
    fmt.Println(index) //1
}
```

## sort.Search原理
```func Search(n int, f func(i int) bool) int``` Search使用二分查找来查找并返回在[0,n)中```f(i)```为真时的最小索引```i```。   
Search的源码十分简单易懂：若当前索引值满足预设的条件，保留右边界，否则对左边界进行缩进。
```go
func Search(n int, f func(int) bool) int {
	// Define f(-1) == false and f(n) == true.
	// Invariant: f(i-1) == false, f(j) == true.
	i, j := 0, n
	for i < j {
		h := int(uint(i+j) >> 1) // avoid overflow when computing h
		// i ≤ h < j
		if !f(h) {
			i = h + 1 // preserves f(i-1) == false
		} else {
			j = h // preserves f(j) == true
		}
	}
	// i == j, f(i-1) == false, and f(j) (= f(i)) == true  =>  answer is i.
	return i
}
```
仔细观察，代码实现要求数组对于```f```这个判断条件来说是单调的，单调指[0,n)中所有索引idx,```idx∈[0,i)```满足```f(idx)=false```,```idx∈[i,n)```满足```f(idx)=true```。
而[2,4,5]查找元素2，因为对于数组[2,4,5],```f func(i int) bool {nums[i]==2}```并不满足单调性，所以得到的结果不满足预期。
![binary_search.drawio.png](/posts/binary_search.drawio.png)
所以，**在使用sort.Search进行二分查找时，对于升序数组查找，需要使用```>=```条件，对于降序数组查找，需使用```<=```条件。    
sort.Search不适用于有序数组查找某个元素，只适用于有序数组中查找满足某一条件的第一个元素的位置，如果想查找特定元素，sort.SearchInts是个好选择**。
```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	a := []int{1, 2, 3, 4, 6, 7, 8}

	x := 2
	i := sort.SearchInts(a, x) 
	// found 2 at index 1 in [1 2 3 4 6 7 8]
	fmt.Printf("found %d at index %d in %v\n", x, i, a)

	x = 5
	i = sort.SearchInts(a, x)
	// found 2 at index 1 in [1 2 3 4 6 7 8]
	fmt.Printf("%d not found, can be inserted at index %d in %v\n", x, i, a)
}
```

## 参考
golang sort package https://pkg.go.dev/sort#Search    
golang: sort.Search can't find first element in a slice https://stackoverflow.com/questions/28344757/golang-sort-search-cant-find-first-element-in-a-slice