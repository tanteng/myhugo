---
title: MySQL 大表加字段思路
author: 小谈
type: post
date: 2017-01-19T12:58:04+00:00
url: /2017/01/mysql-alter-table-big-data/
categories:
  - MySQL

---
给 MySQL 一张表加字段执行如下 sql 就可以了：

<code class="lang:mysql decode:true">ALTER TABLE tbl_tpl ADD title(255) DEFAULT '' COMMENT '标题' AFTER id;</code>

但是线上的一张表如果数据量很大呢，执行加字段操作就会锁表，这个过程可能需要很长时间甚至导致服务崩溃，那么这样操作就很有风险了。

<!--more-->

那么，给 MySQL 大表加字段的思路如下：

① 创建一个临时的新表，首先复制旧表的结构（包含索引）

create table new\_table like old\_table;

② 给新表加上新增的字段

③ 把旧表的数据复制过来

insert into new\_table(filed1,filed2&#8230;) select filed1,filed2,&#8230; from old\_table

④ 删除旧表，重命名新表的名字为旧表的名字

不过这里需要注意，执行第三步的时候，可能这个过程也需要时间，这个时候有新的数据进来，所以原来的表如果有字段记录了数据的写入时间就最好了，可以找到执行这一步操作之后的数据，并重复导入到新表，直到数据差异很小。不过还是会可能损失极少量的数据。

所以，如果表的数据特别大，同时又要保证数据完整，最好停机操作。

另外的方法：

1.在从库进行加字段操作，然后主从切换

2.使用第三方在线改字段的工具

一般情况下，十几万的数据量，可以直接进行加字段操作。