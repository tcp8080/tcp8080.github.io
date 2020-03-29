---
layout: post
title:  "[go tips] how to use slice map channel"
date:   2020-03-29 now
categories: go
---

- make用于内建类型（map、slice 和channel）的内存分配，返回的是这三个对象的引用类型
- new用于各种类型的内存分配，返回的是类型在内存中的初始指针，我们很少直接用new，而是用a:= T{}


func make([]T, len, cap) []T 
//表示创建一个动态的slice，元素类型为T，长度为len，最大容量为cap，cap不填=len, 其返回值是一个三元组，第一项指向底层T数据列的指针，第二项是长度，第三项是容量
```go
type slice struct {
    array unsafe.Pointer
    len   int
    cap   int
}

b := []byte{'g', 'o', 'l', 'a', 'n', 'g'}
// b[1:4] == []byte{'o', 'l', 'a'}, sharing the same storage as b
// b[:2] == []byte{'g', 'o'}
// b[2:] == []byte{'l', 'a', 'n', 'g'}
// b[:] == b
//b[n:m],表示一个从b[n]到b[m-1]的slice，如果n默认为0，m默认为slice长度
//如果使用a := b[n:m], 则a里的数据不会复制一份，而是只把b里面对应的T[n]的指针复制过来
```

### 因为map/slice/channel里都有指针，所以在使用前必须先给它们分配内存空间。
在go里，特意为这三个内建类型创造了一个make方法，用来初始化。

以下是几个初始化的例子。

```go
/**** slice ****/
//创建一个初始元素长度为5的数组切片，元素初始值为0： 
mySlice1 := make([]int, 5) 
//创建一个初始元素长度为5的数组切片，元素初始值为0，并预留10个元素的存储空间： 
mySlice2 := make([]int, 5, 10) 
//切片字面量创建长度为5容量为5的切片,需要注意的是 [ ] 里面不要写数组的容量，因为如果写了个数以后就是数组了，而不是切片了。
mySlice3 := []int{10,20,30,40,50}

/***** map *****/
//创建了一个键类型为string、值类型为PersonInfo
myMap = make(map[string] PersonInfo) 
//也可以选择是否在创建时指定该map的初始存储能力，创建了一个初始存储能力为100的map.
myMap = make(map[string] PersonInfo, 100) 
//创建并初始化map的代码.
myMap = map[string] PersonInfo{ 
  "1234": PersonInfo{"1", "Jack", "Room 101,..."}, 
} 

/******** channel ********/
//创建有缓存通道
ch := make(chan int, 10)
//创建无缓存通道
ch := make(chan int)
defer close(c)
```

一个无缓存ch的使用例子
```go

c := make(chan int)
defer close(c)
go func() { c <- 3 + 4 }()
i := <-c   //只有当i开始接收c的时候，上一行的3+4发送到c才会执行；如果是有缓存的ch，则上一行send会直接执行
fmt.Println(i)
```