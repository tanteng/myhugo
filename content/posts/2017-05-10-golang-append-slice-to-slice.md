---
title: Golang append slice to slice
author: 深 呼吸
type: post
date: 2017-05-10T07:11:22+00:00
url: /2017/05/golang-append-slice-to-slice/
categories:
  - Develop
  - Go

---
把一个 slice 追加到另一个 slice，以下两种方式都是把 s1 追加到 s0 中，但两种结果不同，注意区别。

片段1：

<code class="lang:go decode:true">s0 := []interface{}{1, 100, 122}
s1 := []interface{}{3, 4, 5, 6}
s0 = append(s0, s1)
fmt.Println("result:", s0) //result: [1 100 122 [3 4 5 6]]</code>

此时 s0 结果：result: [1 100 122 [3 4 5 6]]

<!--more-->

片段2：

<code class="lang:go decode:true">s0 := []interface{}{1, 100, 122}
s1 := []interface{}{3, 4, 5, 6}
s0 = append(s0, s1...)
fmt.Println("result:", s0) //result: [1 100 122 3 4 5 6]</code>

此时 s0 结果：result: [1 100 122 3 4 5 6]

s1&#8230; 可变参数类型就是把每个元素加入，否则就是把整体作为切片加入。