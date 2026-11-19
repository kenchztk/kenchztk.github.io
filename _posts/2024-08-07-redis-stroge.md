---
layout: post
title:  redis 底层存储结构
date:   2024-08-07 09:17:20
categories: 架构
tags: redis
---

* content
{:toc}

Redis提供了一系列散列相关的命令给用户使用，例如hset、hget等。然而，这些散列指令对应的底层value的存储并不总是散列表，Redis提供了ziplist以及散列表（hashtable）两种方案进行存储。

当需要存储的key-value结构同时满足下面两个条件时，采用ziplist作为底层存储，否则需要转换为散列表存储。值得注意的是，ziplist的存储顺序与插入顺序一致，而散列表的存储则不一致。

- 1）key-value结构的所有键值对的字符串长度都小于hash-max-ziplist-value（默认值64），该值可以通过配置文件配置。

- 2）散列对象保存的键值对的个数（一个键值对记为1个）小于hash-max-ziplist-entries（默认值512），该值也可以通过配置文件配置。