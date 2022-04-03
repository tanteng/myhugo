---
title: 如何理解 Golang 的参数传递都是值传递？
author: 小谈
type: post
date: 2017-11-19T12:22:01+00:00
url: /2017/11/golang-parameter-passing/
categories:
  - Develop
  - Go

---
在 Golang 中函数之间传递变量时总是以值的方式传递的，无论是 int,string,bool,array 这样的内置类型（或者说原始的类型），还是 slice,channel,map 这样的引用类型，在函数间传递变量时，都是以值的方式传递，也就是说传递的都是值的副本。

<!--more-->

### 内置类型参数传递

内置类型传递的时候是值的副本，这个好理解，随便举个例子：

```package main

import (
	"fmt"
)

func main() {
	num := 10
	num2 := increase(num, 10)
	fmt.Println(num2)
}

func increase(num int, add int) int {
	return num + add
}
```

这里 num 传入 increase 函数，是拷贝值的副本，并且返回一个新的值。假设 num 是一个很大的数组，那么传递给函数的就是这个很大数组的拷贝。（这样很浪费内存，真实情况如果要传一个很大的数组，应该传递数组的指针）

### 引用类型的参数传递

引用类型的参数传递也是值的拷贝。

例子：

```package main

import (
	"fmt"
)

func main() {
	slice1 := []string{"zhang", "san"}
	modify(slice1)
	fmt.Println(slice1)
}

func modify(data []string) {
	data = nil
}
```

运行结果：

[zhang san]

这个例子证明了作为引用类型的切片，参数传递不是传的引用，而是传的值，如果是传的引用，那么函数对它的修改会受到影响，而这里切片内容并没有改变成 nil.

但是有一个例子比较误导人，我们看一看：

```package main

import (
	"fmt"
)

func main() {
	slice1 := []string{"zhang", "san"}
	modify(slice1)
	fmt.Println(slice1)
}

func modify(data []string) {
	data[1] = "si"
}
```

运行结果：

[zhang si]

这里为什么改变了切片的内容呢？

#### 什么是标头？

搞清楚这个问题，首先要知道什么是“标头”这个概念？引用《Go语言实践》中的一段话：

> Go 语言里的引用类型有如下几个：切片、映射、通道、接口和函数类型。**当声明上述类型的变量时，创建的变量被称作标头（header）值。**从技术细节上说，字符串也是一种引用类型。每个引用类型创建的标头值是包含一个指向底层数据结构的指针。**因为标头值是为复制而设计的，所以永远不需要共享一个引用类型的值。标头值里包含一个指针，因此通过复制来传递一个引用类型的值的副本，本质上就是在共享底层数据结构。**

_总而言之，引用类型在函数传递的时候，是值传递，只不过这里的“值”指的是标头值。_

我们分别打印这个切片变量传参前后的指针地址，和传参前后切片中元素的指针地址：

```package main

import (
	"fmt"
)

func main() {
	slice1 := []string{"zhang", "san"}
	fmt.Printf("%p\n", &slice1)
	fmt.Printf("%p\n", &slice1[1])
	modify(slice1)
	fmt.Println(slice1)
}

func modify(data []string) {
	fmt.Printf("%p\n", &data)
	fmt.Printf("%p\n", &data[1])
	data[1] = "si"
}
```

运行结果：

0xc42000a060
  
0xc42000a090
  
0xc42000a0a0
  
0xc42000a090

这再次证明了切片传递的不是指针地址，因为变量前后地址不同。

这也证明了切片的参数传递的是传值的形式，具体是传标头值的拷贝，因为指向元素的指针地址相同。