---
layout: post
title:  rst 包的一种情形
date:   2026-08-25 10:59:09
categories: Network
tags: rst
---

* content
{:toc}

当服务器close一个连接时，若client端接着发数据。根据TCP 协议的规定，会收到一个RST响应，client再往这个服务器发送数据时，系统会发出一个SIGPIPE信号给进程，告诉进程这个连接已经断开了，不要再写了。 

根据信号的默认处理规则SIGPIPE信号的默认执行动作是terminate(终止、退出),所以client会退出。若不想客户端退出可以把 ``SIGPIPE`` 设为 ``SIG_IGN``

如:  ``signal(SIGPIPE,SIG_IGN); ``

这时SIGPIPE交给了系统处理。 
服务器采用了fork的话，要收集垃圾进程，防止僵尸进程的产生，可以这样处理： 

``signal(SIGCHLD,SIG_IGN); ``交给系统init去回收。 

这里子进程就不会产生僵尸进程了。 

 