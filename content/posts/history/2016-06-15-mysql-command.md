---
title: 常用Linux操作数据库命令及MySQL语句
author: 深 呼吸
type: post
date: 2016-06-15T11:29:06+00:00
url: /2016/06/mysql-command/
duoshuo_thread_id:
  - 6307512300450349826
categories:
  - Develop
  - MySQL
  - 数据库

---
以下是在Linux下经常会用到的MySQL的一些命令，导出，导入，建库建表，备份，以及MySQL修改字段，添加字段等语法。

### 数据库表导入

恢复sql到数据库，不会覆盖数据库，仅执行恢复的sql语句，常用于数据库表的导入：

<code class="lang:mysql decode:true ">mysql -uroot -p tanteng.me  &lt; mobile_promote.sql</code>

<!--more-->

### 数据库表导出

#### 导出整个数据库

<code class="lang:mysql decode:true ">mysqldump -u root -p tanteng.me &gt; www.sql</code>

#### 导出数据库表

支持以空格形式分隔多张表。

<code class="lang:mysql decode:true ">mysqldump -u root -p tanteng.me options &gt; options.sql</code>

#### 查询数据导出到txt文件

<code class="lang:mysql decode:true">mysql -u root -p -e"select id,username from walle.user" &gt; user.txt</code>

#### 导出特定条件的 sql 语句

<code class="lang:mysql decode:true ">mysqldump -u root -p jiuwo_op v_block --where='blockid in(64,65)' &gt; block.sql</code>

### MySQL增加修改字段

#### 增加字段

<code class="lang:mysql decode:true">ALTER TABLE js_business_bank_info ADD bbi_ramdom char(100) DEFAULT '' COMMENT '生成的随机银行备注' AFTER bbi_source;</code>

#### 修改字段

修改字段类型：

<code class="lang:mysql decode:true">ALTER TABLE supplier MODIFY supplier_name char(100) NOT NULL;</code>

修改字段名称：

<code class="lang:mysql decode:true">ALTER TABLE new_mobile_promote CHANGE `promote_link` `link` varchar(255);</code>

修改字段注释

<code class="lang:mysql decode:true">ALTER TABLE t1 MODIFY col1 BIGINT UNSIGNED DEFAULT 1 COMMENT 'my column';</code>

删除字段

<code class="lang:mysql decode:true">ALTER TABLE new_mobile_promote DROP COLUMN list_image_url;</code>

添加自动更新时间字段

<code class="lang:mysql decode:true">ALTER TABLE `civil_vote` ADD `created_at` TIMESTAMP ON UPDATE CURRENT_TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ;</code>

### MySQL建库

新建数据库并设定utf8编码

<code class="lang:mysql decode:true ">CREATE DATABASE mydb DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;</code>

删除数据库

<code class="lang:mysql decode:true ">DROP DATABASE db_name</code>

### 添加唯一索引

<code class="lang:mysql decode:true ">ALTER TABLE `table_name` ADD UNIQUE (`column`)</code>

插入如果唯一索引字段存在则执行更新操作：

<code class="lang:mysql decode:true ">INSERT INTO table (a,b,c) VALUES (1,2,3) ON DUPLICATE KEY UPDATE c=c+1;</code>

or

<code class="lang:mysql decode:true ">INSERT INTO table (a,b,c) VALUES (1,2,3) ON DUPLICATE KEY UPDATE c=VALUES(c);</code>

### 复制表

基于 orig_tbl 创建一张空表，包含所有字段和索引。比如用于按天分表，需要用到按天建表的场景。

<code class="lang:mysql decode:true">CREATE TABLE new_tbl LIKE orig_tbl;</code>

### 复制一张表数据到另一张表

两种情况：

① 新表的结构和旧表一样

<code class="lang:mysql decode:true ">INSERT INTO 新表 SELECT * FROM 旧表</code>

② 新表结构和旧表不一样，可能新表加了一些字段的情形

<code class="lang:mysql decode:true ">INSERT INTO 新表(字段1,字段2,…….) SELECT 字段1,字段2,…… FROM 旧表</code>

比如这个场景，会用到复制表结构，以及复制一张表数据到另一张表数据的方法，当一张表数据很大的时候，如果线上加字段会导致锁表，很长时间没有响应，影响用户的操作，甚至停止业务的正常进行，这个时候可以新建一张表，增加字段，把旧表数据复制到新表，最后删掉旧表，把新表重命名为旧表的名字。不过注意这段时间可能有新增的数据，要把新增的数据也导入到新表。

### 其他

显示建表 sql：

<code class="lang:mysql decode:true">show create table table_name;</code>

显示自增长起始和递增值：

<code class="lang:mysql decode:true">SHOW VARIABLES LIKE 'auto_inc%';</code>

修改 auto\_increment\_increment 值：

<code class="lang:mysql decode:true ">SET @@auto_increment_increment=1;</code>

示例：

<code class="lang:mysql decode:true ">mysql&gt; SHOW VARIABLES LIKE 'auto_inc%';
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| auto_increment_increment | 2     |
| auto_increment_offset    | 2     |
+--------------------------+-------+</code>

&nbsp;