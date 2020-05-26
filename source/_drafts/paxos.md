---
title: paxos
toc: true
tags:
---
Paxos Made Simple

## Basic Paxos
  角色
    * Client/proposer
      负责提案并执行提案
    * Coordinator:Proposer协调者  
    * Acceptor:负责对Proposal进行投票表决
    * Leader:众多Coordinator中指定一个作为Leader
    * Learner
  问题定义：
    - 决议(value)只有在被proposers提出后才被批准
    - 在一个Paxos算法实例执行中，只能批准一个value
    - learners只能获得被批准(chosen)的value
  约束：
    - P1: 一个 acceptor 必须接受（accept）第一次收到的提案。
    - P2: 一旦一个具有 value v 的提案被批准（chosen），那么之后批准（chosen）的提案必须具有 value v。
      - P2a：一旦一个具有 value v 的提案被批准（chosen），那么之后任何 acceptor 再次接受（accept）的提案必须具有 value v。
      - P2b：一旦一个具有 value v 的提案被批准（chosen），那么以后任何 proposer 提出的提案必须具有 value v。
      - P2c：如果一个编号为 n 的提案具有 value v，那么存在一个多数派，要么他们中所有人都没有接受（accept）编号小于n的任何提案，要么他们已经接受（accept）的所有编号小于 n 的提案中编号最大的那个提案具有 value v。
      通过一个决议分为两个阶段:
   两阶段：
     * prepare阶段：
       proposer选择一个提案编号n并将prepare请求发送给acceptors中的一个多数派；acceptor到prepare消息后，如果提案的编号大于它已经回复的所有prepare消息(回复消息表示接受accept)，则acceptor将自己上次接受的提案回复给proposer，并承诺不再回复小于n的提案；
     * Accept阶段：
       当一个proposer收到了多数acceptors对prepare的回复后，就进入批准阶段。它要向回复prepare请求的acceptors发送accept请求，包括编号n和根据P2c决定的value（如果根据P2c没有已经接受的value，那么它可以自由决定value）。在不违背自己向其他proposer的承诺的前提下，acceptor收到accept请求后即批准这个请求。

## Multi Paxos
   * 选举: multi-paxos需要选主，但是不要求新leader包含所有的日志；
   * 恢复: 对未confirm 的proposal逐条进行paxos
       http://oceanbase.org.cn/?p=111
   * 日志空洞: 新任leader在开始执行重确认前，需要先知道重确认的结束位置，因为leader本地相对于集群内多数派可能已经落后很多日志，所以需要想集群内其他server发送请求，查询每个server本地的最大logID，并从多数派的应答中选择最大的logID作为重确认的结束位置。也即开始提供服务后写日志的起始logID。     
   * leader的乱序提交log

## Fast Paxos
