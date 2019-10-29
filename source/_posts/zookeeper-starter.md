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
## ZK集群模式（QuorumPeerMain）
Zookeeper集群模式的入口类是QuorumPeerMain，主要完成两件事：配置解析及节点初始化并启动(QuorumPeer)。
梳理配置解析对理解Zookeeper的整体实现有很大的帮组，比如Zookeeper使用了哪几类端口，作用分别是什么？
### 配置解析
配置类QuorumPeerConfig里的配置项主要包含：
* 节点ID: sid
* 集群节点的IP及开放的端口(QuorumServer)
配置文件中相关的配置，解析类(QuorumMaj)如下
```
clientPort=2181
server.1=hostname1:2888:3888
server.2=hostname2:2888:3888
server.3=hostname3:2888:3888
server.4=hostname4:2888:3888:observer

```
从QuorumMaj 中可以看出：
- 2181 监听客户端请求
- 2888 集群内部通信（复制）
- 3888 集群内部选举
- observer 节点4是观察者，默认为PARTICIPANT（leader or follower）

### QuorumPeer 启动

## Leader选举
未完，待续
## 崩溃恢复（数据同步）
未完，待续
## 原子广播
未完，待续
