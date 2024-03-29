---
title: Golang slice
date: 2023-01-05
tags: [Golang]
---


### 数组

数组是由相同类型元素的集合组成的数据结构, 计算机会为数组分配一块连续的内存来保存其中的元素

数组是定长的, 必须指定长度或满足编译器对数组大小的推导

```go

arr1 := [3]int{1, 2, 3}
arr2 := [...]int{1, 2, 3}

```

### 切片

切片类型的声明方式与数组有一些相似, 不过由于切片的长度是动态的, 所以声明时只需要指定切片中的元素类型


数据结构

```go
type SliceHeader struct {
	Data uintptr
	Len  int
	Cap  int
}
```
- Data 是指向数组的指针;

    Data 是一片连续的内存空间, 这片内存空间可以用于存储切片中的全部元素, 数组中的元素只是逻辑上的概念, 底层存储其实都是连续的, 所以我们可以将切片理解成一片连续的内存空间加上长度与容量的标识
- Len 是当前切片的长度；
- Cap 是当前切片的容量, 即 Data 数组的大小,

程序在运行区间可以修改它的长度和范围。当切片底层的数组长度不足时就会触发扩容, 切片指向的数组可能会发生变化, 不过在上层看来切片是没有变化的, 上层只需要与切片打交道不需要关心数组的变化

#### 扩容
在分配内存空间之前需要先确定新的切片容量, 运行时根据切片的当前容量选择不同的策略进行扩容：

- 如果期望容量大于当前容量的两倍就会使用期望容量；
- 如果当前切片的长度小于 1024 就会将容量翻倍；
- 如果当前切片的长度大于 1024 就会每次增加 25% 的容量, 直到新容量大于期望容量；

#### copy

```go

func slicecopy(to, fm slice, width uintptr) int {
	if fm.len == 0 || to.len == 0 {
		return 0
	}
	n := fm.len
	if to.len < n {
		n = to.len
	}
	if width == 0 {
		return n
	}
	...

	size := uintptr(n) * width
	if size == 1 {
		*(*byte)(to.array) = *(*byte)(fm.array)
	} else {
        // runtime.memmove 将整块内存的内容拷贝到目标的内存区域中
		memmove(to.array, fm.array, size)
	}
	return n
}
```