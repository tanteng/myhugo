---
title: Golang 中”泛型”的支持
author: 深 呼吸
type: post
date: 2017-08-11T10:14:10+00:00
url: /2017/08/golang-generic/
categories:
  - Develop
  - Go

---
Golang不支持一般的类似java中的标记式泛型。很多人因此而十分不满，认为没有泛型增加了很多工作量。而目前由于泛型支持的复杂性，Golang的设计和实现者并没有把这个泛型支持作为紧急需要增加的特性。但是，如果真的没有泛型，就一定不行了么？答案当然是否定的。没有泛型也可以，而且我觉得代码更简单，直接，有趣（个人观点，勿喷）。

我们这里打算以一些例子来讲解Golang中如何处理这个问题。

首先，我们看一个冒泡排序的问题。针对整型数组切片的排序。

```php
package main

import (
	"fmt"
)

func bubbleSort(array []int) {
	for i := 0; i &lt; len(array); i++ {
		for j := 0; j &lt; len(array)-i-1; j++ {
			if array[j] > array[j+1] {
				array[j], array[j+1] = array[j+1], array[j]
			}
		}
	}
}

func main() {
	a1 := []int{3, 2, 6, 10, 7, 4, 6, 5}
	bubbleSort(a1)
	fmt.Println(a1)
}
```

上面的例子输出为：

<code class="lang:go decode:true ">[2 3 4 5 6 6 7 10]</code>

那么，我们如果希望这个bubbleSort能够同时支持float类型数据排序，或者是按照字符串的长度来排序应该怎么做呢？在其他的例如java语言中，我们可以将bubbleSort定义为支持泛型的排序，但是Go里面就不行了。为了达到这个目的，我们可以使用interface来实现相同的功能。

针对上面的排序问题，我们可以分析一下排序的步骤：

  1. 查看切片长度，以用来遍历元素(Len)；
  2. 比较切片中的两个元素(Less)；
  3. 根据比较的结果决定是否交换元素位置(Swap)。
  
    到这里，或许你已经明白了，我们可以把上面的函数分解为一个支持任意类型的接口，任何其他类型的数据只要实现了这个接口，就可以用这个接口中的函数来排序了。

```php
type Sortable interface{
	Len() int
	Less(int, int) bool
	Swap(int, int)
}
```

下面，我们就用几个例子来描述一下这个接口的用法。

```
package main

import (
	"fmt"
)

type Sortable interface {
	Len() int
	Less(int, int) bool
	Swap(int, int)
}

func bubbleSort(array Sortable) {
	for i := 0; i &lt; array.Len(); i++ {
		for j := 0; j &lt; array.Len()-i-1; j++ {
			if array.Less(j+1, j) {
				array.Swap(j, j+1)
			}
		}
	}
}

//实现接口的整型切片
type IntArr []int

func (array IntArr) Len() int {
	return len(array)
}

func (array IntArr) Less(i int, j int) bool {
	return array[i] &lt; array[j]
}

func (array IntArr) Swap(i int, j int) {
	array[i], array[j] = array[j], array[i]
}

//实现接口的字符串，按照长度排序
type StringArr []string

func (array StringArr) Len() int {
	return len(array)
}

func (array StringArr) Less(i int, j int) bool {
	return len(array[i]) &lt; len(array[j])
}

func (array StringArr) Swap(i int, j int) {
	array[i], array[j] = array[j], array[i]
}

//测试
func main() {
	intArray1 := IntArr{3, 4, 2, 6, 10, 1}
	bubbleSort(intArray1)
	fmt.Println(intArray1)

	stringArray1 := StringArr{"hello", "i", "am", "go", "lang"}
	bubbleSort(stringArray1)
	fmt.Println(stringArray1)
}
```

输出结果为：

<code class="lang:go decode:true ">[1 2 3 4 6 10]
[i am go lang hello]</code>

上面的例子中，我们首先定义了一个IntArr类型的整型切片类型，然后让这个类型实现了Sortable接口，然后在测试代码中，这个IntArr类型就可以直接调用Sortable接口的bubbleSort方法了。

另外，我们还演示了一个字符串切片类型StringArr按照字符串长度来排序的例子。和IntArr类型一样，它实现了Sortable即可定义的方法，然后就可以用Sortable即可的bubbleSort方法来排序了。

总结

上面的例子，是一种Golang中支持所谓的“泛型”的方法。这种泛型当然不是真正意义上面的泛型，但是提供了一种针对多种类型的一致性方法的参考实现。

（原文地址：<a href="http://studygolang.com/articles/7699" target="_blank" rel="noopener nofollow">http://studygolang.com/articles/7699</a>）