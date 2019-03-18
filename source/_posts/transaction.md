---
title: Distributed transaction
tags: XA
date: 2019-03-18 23:46:59
---


2PC、3PC
===
分布式事务中必须处理超时、节点(coordinator、participate)宕机、节点宕机恢复（fail-recover）、网络分区等异常情况。[2PC和3PC基本概念][1]

## 基本概念

* 2PC 包含两个阶段：voting和commit
* 3PC 包含三个阶段：voting、pre-commit和commit

## 2PC和3PC的区别
   2PC和3PC最大的区别在于是否[blocking][2],[Why 2PC is blocking][4]?

## 2PC及缺点
   2PC能处理参与者宕机的情况，但无法处理协调者和参与者同时宕机的情况[一致性、2PC和3PC][3]。
* 协调者失败，会blocking参与者，参与者没法单独决定是提交(commit)还是回滚(rollback)

## 3PC及缺点
 3PC引入time-out及pre-commit解决2PC blocking参与者的问题，但仍然无法解决网络分区问题。



[1]: https://pdfs.semanticscholar.org/06e2/5c1f69155e53af51170c08687e1dcf272974.pdf "2PC和3PC的概念"
[2]: https://www.cs.purdue.edu/homes/bb/cs542-11Spr/week10_lecture2-Commit.ppt "Distributed RDBMS"
[3]: https://zhuanlan.zhihu.com/p/21994882 "一致性、2PC和3PC"
[4]: https://roxanageambasu.github.io/ds-class/assets/lectures/lecture16.pdf "Why 2PC is blocking"
