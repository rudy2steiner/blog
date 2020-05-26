---
title: Distributed System Basics
draft: true
toc: true 
date: 2019-10-13 22:23:49
tags: [Distributed theory,CAP定理]
---

  当谈论分布式系统时，我们到底在谈论什么？通常来说，分布式系统是指分布在多个机器上的进程共同组成一个系统，对外提供服务，主要是为了解决Scalable问题，如分布式存储,分布式计算。为了能够使分布式系中的节点能达成一致状态，必须采用分布式算法，如PAXOS、RAFT和ZAB等。这些分布式算法的设计主要涉及几个理论，如FLP，CAP及BASE。

## FLP impossibility

```
  No consensus protocol is totally correct in spite of one fault.
```
大白话就是：在异步系统中，即使只有一个节点故障，也不存在完全正确的共识算法。那Paxos和Raft为什么正确？

### 系统模型
任何分布式算法或者定理，对系统场景的假设，称为系统模型，FLP[1]基于以下假设：

* 异步通信 与同步通信最大的区别是没有时钟、不能时间同步、不能使用超时、不能探测进程失败
* 通信健壮  消息会被延迟，但最终会送达；并且消息仅送达一次（不重复）
* fail-stop模型 进程失败如同宕机
* 最多只允许一个进程失败

### Consensus 定义

1. termination: 所有进程最终会在有限步数中结束并选取一个值，算法不会无限运行下去
2. agreement: 所有进程必须同意同一个值
3. validity: 最终达成一致的值必须是V1到Vn其中之一，如果所有的初始值是Vx,那么最终结果必须是Vx

完全正确（totally correct）是指同时满足safety和liveness[2]。FLP是想告诉大家不要浪费时间去为异步分布式系统设计在任意场景下都能实现共识的算法。


## CAP[3]定理

一个分布式系统最多只能同时满足一致性（Consistency）、可用性(Availability)和分区容错性(Partition tolerance)这三项中的两项。

* Consistency 一致性即所有节点在同一时间完全一致（强一致）。
  - 通常我们说RAFT、PAXOS和ZAB都是强一致的，如果读写都在主上，则从客户端来看是强一致。
  - 采用Quorum的共识算法，从服务端来看，都是最终一致。

* Availability 可用性即非故障节点在合理的时间内返回合理的响应。
* Partition tolerance 分区容错性即某节点发生网络分区时，仍能对外提供满足一致性或可用性的服务。

## BASE理论

BASE是指基本可用(Basically Available)、软状态(Soft State)、最终一致性(Eventually Consistency)。

* 基本可用（Basically Available）
  在出现故障时候，允许损失部分可用性，即保证核心可用。
* 软状态(Soft State)
  允许系统存在中间状态，而该中间状态不影响系统整体可用性。
* 最终一致性(Eventually Consistency)
  所有副本经过一定的时间后，最终能够达到一致的状态。


## 参考文献

[1]. [Impossibility of Distributed Consensus with One Faulty](http://citeseerx.ist.psu.edu/viewdoc/download;jsessionid=D48A23891CFFED4A69D7B546041F97EC?doi=10.1.1.43.8770&rep=rep1&type=pdf)
[2]. [safety-and-liveness-properties-a-survey](https://lrita.github.io/images/posts/distribution/safety-and-liveness-properties-a-survey.pdf)
[3]. [Lynch, Nancy, and Seth Gilbert. “Brewer's conjecture and the feasibility of consistent, available, partition-tolerant web services.” ACM SIGACT News, v. 33 issue 2, 2002, p.51-59](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.67.6951&rep=rep1&type=pdf)                                     
