---
title: MySQL查看数据库大小
author: 深 呼吸
type: post
date: 2016-06-03T11:24:42+00:00
url: /2016/06/mysql-data-length/
duoshuo_thread_id:
  - 6291925586281497346
categories:
  - Develop
  - MySQL
  - 数据库

---
1、进入information_schema 数据库（存放了其他的数据库的信息）
  
use information_schema;

2、查询所有数据的大小：
  
select concat(round(sum(data_length/1024/1024),2),&#8217;MB&#8217;) as data from tables;

<!--more-->

3、查看指定数据库的大小：
  
比如查看数据库home的大小
  
select concat(round(sum(data\_length/1024/1024),2),&#8217;MB&#8217;) as data from tables where table\_schema=&#8217;home&#8217;;

4、查看指定数据库的某个表的大小
  
比如查看数据库home中 members 表的大小
  
select concat(round(sum(data\_length/1024/1024),2),&#8217;MB&#8217;) as data from tables where table\_schema=&#8217;home&#8217; and table_name=&#8217;members&#8217;;