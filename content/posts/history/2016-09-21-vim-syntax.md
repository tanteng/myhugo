---
title: vim 常用操作
author: 深 呼吸
type: post
date: 2016-09-21T14:06:02+00:00
url: /2016/09/vim-syntax/
categories:
  - Linux

---
vim 在 Linux 下使用很多，但是习惯了在 Windows 下的文本操作，在 vim 中进行文本操作会觉得很不方便，但是 vim 是一个很强大的工具，只是还不熟练去使用它，下面是一些常用的 vim 文本操作方法。

<!--more-->

### vim 撤销和恢复操作

在不可编辑模式下，使用 u 即可撤销上一次操作，使用 Ctrl+r 恢复上一次操作。

### vim 区块选择和复制粘贴

vim 进入某个文件，按 v，进入 VISUAL 模式，使用 h,j,k,l 或者方向键移动光标即可选中内容，按 y 完成复制，在需要粘贴的地方按 p 完成粘贴。

按 v 进入 VISUAL 模式，再次按 v 即可退出 VISUAL 模式。如果复制了内容，也可以按 i 进入编辑模式进行编辑操作，再 esc 退出来使用 p 完成粘贴。

### vim 移动到开头和末尾

#### 文档开头和末尾

gg:命令将光标移动到文档开头

G:命令将光标移动到文档末尾

#### 一行的开头和末尾

0 移到一行开头

$ 移到一行末尾

### vim 翻页

相当于 page up 和 page down 的效果。

<p class="">
  Ctrl+f 往前滚动一整屏
</p>

<p class="">
  Ctrl+b 往后滚动一整屏
</p>

<p class="">
  Ctrl+d 往前滚动半屏
</p>

<p class="">
  Ctrl+u 往后滚动半屏
</p>

### vim 删除一行或多行

dd   删除一行
  
ndd 删除以当前行开始的n行

### vim 复制多行

<p class="">
  ：9，15 copy 16  或 ：9，15 co 16
</p>

<p class="">
  由此可有：
</p>

<p class="">
  ：9，15 move 16  或 :9,15 m 16 将第9行到第15行的文本内容到第16行的后面
</p>