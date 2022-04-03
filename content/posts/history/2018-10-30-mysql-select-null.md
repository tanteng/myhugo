---
title: MySQL 字段为 NULL 的查询问题
author: 深 呼吸
type: post
date: 2018-10-30T03:29:00+00:00
url: /2018/10/mysql-select-null/
description: Mysql难以优化引用可空列查询，它会使索引、索引统计和值更加复杂。可空列需要更多的存储空间，还需要mysql内部进行特殊处理。可空列被索引后，每条记录都需要一个额外的字节，还能导致MYisam 中固定大小的索引变成可变大小的索引。
categories:
  - Develop
  - MySQL

---
假设 MySQL 中某数据表 code 字段是可为 NULL

```mysql
... AND code NOT IN () ...
... AND (code IS NULL or code NOT IN()) ...
```

是不一样的，第一个查不出 code IS NULL 的数据。

> <a href="https://stackoverflow.com/questions/3536670/mysql-selecting-rows-where-a-column-is-null" target="_blank" rel="noopener nofollow">https://stackoverflow.com/questions/3536670/mysql-selecting-rows-where-a-column-is-null</a>
> 
> SQL NULL&#8217;s special, and you have to do WHERE field IS NULL, as NULL cannot be equal to anything,
> 
> including itself (ie: NULL = NULL is always false).
> 
> See Rule 3 <a href="https://en.wikipedia.org/wiki/Codd%27s_12_rules" target="_blank" rel="noopener nofollow">https://en.wikipedia.org/wiki/Codd%27s_12_rules</a>

几个不用 NULL 的理由：

1.NULL 和 “” 不相同，比如查询 code != &#8216;xx&#8217; 的记录并不包含 NULL 的，必须加一个条件 and is null. 容易遗漏。

2.Mysql难以优化引用可空列查询，它会使索引、索引统计和值更加复杂。可空列需要更多的存储空间，还需要mysql内部进行特殊处理。可空列被索引后，每条记录都需要一个额外的字节，还能导致MYisam 中固定大小的索引变成可变大小的索引。

—— 出自《高性能mysql第二版》

因此建议除了 timestamps 这种字段，都用 NOT NULL DEFAULT &#8221; 表示。

关于 NULL 和唯一索引，唯一性索引是允许多个 NULL 值的存在的。