---
layout: post
title:  SQL概述
date:   2024-08-06 08:49:47
categories: 数据库
tags: sql
---

* content
{:toc}

SQL是进行关系型数据库操作时通用的查询语言。SQL通常由``DDL（数据定义语言）``和``DML（数据操作语言）``两个部分组成。

简单理解:
- ``DDL(DATA DEFINE LANGUAGR)``是在数据库中存储现实世界中的实体时所用的描述语言，它通常强调数据库对现实实体数据结构的建模，如对数据库、数据库表或视图等进行创建、删除、修改等操作。
- ``DML(Data Manipulation Language)``是对数据库中已存在的实体数据进行相关访问的操作语言，它强调的是对数据进行访问，DML其实就是人们常说的CRUD（Create，Read，Update，Delete）。

初次之外还有``DCL``和``TCL``两种概念
- ``DCL``是``Data Control Language``的缩写，意为数据控制语言，用于授予和回收数据库对象上的权限。典型的DCL有grant和revoke。
- ``TCL``是``Transaction Control Language``的缩写，意为事务控制语言，用于管理DML对数据的改变。它允许一组DML语句联合成一个逻辑事务。典型的TCL有commit、rollback、savepoint、set transaction等。