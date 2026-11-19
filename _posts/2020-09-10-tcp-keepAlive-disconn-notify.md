---
layout: post
title:  "使用长连接之后，客户端、服务端怎么知道本次传输结束呢?"
date:   2020-09-10 19:07:00
categories: Network
tags: network
---

* content
{:toc}

##### 两种方案：
1、判断传输数据是否达到了Content-Length指示的大小；
# 
2、如果动态生成的文件没有Content-Length，它是分块传输（chunked），这时候就要根据chunked编码的数据在最后有一个空chunked块，表明本次传输数据结束。
#
更细节的介绍可以看 [这篇文章](https://www.cnblogs.com/skynet/archive/2010/12/11/1903347.html).