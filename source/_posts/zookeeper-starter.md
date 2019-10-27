---
title: Zookeeper源码分析-概述
date: 2019-10-27 22:24:09
tags: [ZAB,zookeeper]
---
Zookeeper 是一个容错的分布式协调服务通常用于维护配置信息、服务发现、Leader选举等场景。采用ZAB(Zookeeper atomic broadcast)共识算法使得服务集群高可用，例如一个5节点的服务集群最多可以允许2个节点不可用(宕机、网络隔离)，集群仍能对外提供服务。

ZAB协议主要有四个状态(ZabState)：
* ELECTION
* DISCOVERY
* SYNCHRONIZATION
* BROADCAST

Zookeeper 集群中节点有四种角色(ServerState)：
* LOOKING (初始角色)
* FOLLOWING（follower）
* LEADING (leader)
* OBSERVING (observer)

理解zookeeper的实现主可以顺着集群中节点角色及协议状态的转化过程，梳理出大致的。Zookeeper有单例和集群两种模式,本文主要从实现的角度介绍Zookeeper集群模式的程序入口、选举、崩溃恢复、广播等内容。
## ZK入口
未完，待续
## Leader选举
未完，待续
## 崩溃恢复（数据同步）
未完，待续
## 原子广播
未完，待续
