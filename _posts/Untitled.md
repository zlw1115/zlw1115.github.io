---
layout:     post
title:      SQL语法相关
subtitle:   
date:       2023-12-11
author:     Zeng Liwen
header-img: 
catalog: 	 true
tags:
    - work	
---

[toc]

### sql语句，相关语法记录

参考博客：



[SQL学习笔记 - CASE WHEN THEN - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/165423831)

[SQL With As 用法 - Niko12230 - 博客园 (cnblogs.com)](https://www.cnblogs.com/Niko12230/p/5945133.html)

#### 一、背景

在检查软件包依赖关系时，借助数据库，使用sql查询找出软件包之间的联系。如下的sql编写：

![](E:\NOTE!!!!\zlw1115.github.io\img\2023-12-11\依赖查找sql语句.jpg)

#### 二、语法总结

##### 1. UNION操作符

` SELECT pkgKey FROM {table} WHERE name = '{name}' UNION SELECT pkgKey from files WHERE name = '{name}'`

UNION操作符用于合并两个或多个SELECT语句的结果集。UNION内部的每个SELECT语句必须拥有相同数量的列。列也必须拥有相似的数据类型，同时，每个SELECT语句中的列的顺序必须相同。

```
SELECT column_name(s) FROM table1
UNION
SELECT column_name(s) FROM table2;
# UNION操作符选取不用的值，如果允许重复的值，需要使用UNION ALL
SELECT column_name(s) FROM table1
UNION ALL
SELECT column_name(s) FROM table2;
```

- UNION结果集中的列名总是等于UNION中第一个SELECT语句中的列名。