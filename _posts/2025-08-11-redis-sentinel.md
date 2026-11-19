---
layout: post
title:  Redis Sentinel 高可用架构
date:   2025-08-11 16:46:15
categories: 架构
tags: redis
---

* content
{:toc}

1、建立sentinel集群，有一个leader角色

2、官网推荐3主3从 外加 3个sentinel(sentinel集群，内部使用raft自选举)

3、keepAlived:（每个sentinel 1次/秒频率向 master、salve及其他sentinel实例发送PING命令健康检查）


主动下线: Sentinel Leader 单向的同 Master 产生通信超时（不可靠，可能是为Sentinel Leader 和 Master 网络通信不畅，但其实 Master 是可以正常的提供服务的）

客观下线：主观下线后，Sentinel Leader 通知其他的Sentinel节点向Master发起PING请求，多个 sentinel PING 不通（多个=总数除以2+1），则将其标记为客观下线，之后开启故障转移的过程（就算后续异常的master节点恢复还是被降级为slave）

基于全量复制存储 维护上更加简单

为了节省资源，可以将单点slave和单点哨兵部署在一起，master单独一台机器，总共3台机器


