---
title: Raft 算法及实现
date: 2019-12-04 22:52:45
tags:
---
Raft算法包含如下子问题：
   - Leader 选举,成员变更
   - Log replication，log compact
   - Safety

server state:
  - leaeder
  - follower
  - candidate

```
Raft中主要的对象：
Persistent state{
  currentTerm,
  votedFor,
  log[]
}


Volatile state on all servers{
  commitIndex 最大的日志log entry索引
  lastApplied 最大的已应用到状态机的log entry索引
}
Volatile state on leaders{
  nextIndex[]  下一个即将复制给某个follower的log entry索引
  matchIndex[] 已经复制给每一个follower的最大log entry索引
}

RequestVote RPC{
  term 候选者任期
  candidateId 候选者ID
  lastLogIndex 候选者的最后一个log entry 索引
  lastLogTerm  候选者最后一个log entry 的term
}
Vote Request 接受者实现：
 * 回复 false if term < currentTerm
 * 如果没有给任何选选者投票且候选者log不落后于自己的日志（），true
```

遵循的规则
```
 所有Servers遵循的规则：
  * if commitIndex > lastApplied,更新lastApplied并应用日志到状态机
  * if RPC 请求或者响应的 term(T)> currentTerm,更新currentTerm =T，变成follower

 Follower遵循的原则：
  * 响应来之候选者和领导者的RPC
  * 没有收到当前Leader AppendEnties或者给候选人投票并超时，则变成候选人

 候选人遵循的规则：
  * 在和候选人交流和启动选举时，
   - currentTerm自增
   - 投票给自己
   - 重置election timer
   - 发送 RequestVote RPCs给所有的server
  * 获得大多数选票，变为leader
  * 收到新Leader的AppendEnties RPC，变成follower
  * 如果选举超时，开启新的选举
Leader 遵循的原则：
  *  一旦成为Leader, 发送心跳给所有的server，防止选举超时
  * 收到client请求，追加entry 到本地日志，等状态机应用以后再回复client
  * 如果 last log 索引>= nextIndex ,发送以nextIndex开始的AppendEntries RPC
    - 成功，更新follower对应的 nextIndex和matchIndex
    - 由于日志不一致导致失败，则减小nextIndex并重试,找到最近一致log
  * 存在一个N> commit index, 且大多数的matchIndex[i]>=N并且log[N].term==currentTerm
    将commitIndex = N  
```

## 选举约束
* (lastLogTerm,lastLogIndex) 而元组比较
  - up-to-date: 保证包含所有已提交的log entry
## log 复制和提交约束
* 不能依靠绝大多数策略提交非自己任期内的log entry？
  会出现log 覆盖的情况，https://juejin.im/post/5ce7be0fe51d45775c73dc57


异常情况：
1. Split选举
2. Leader 或者 follower crash 问题
3. 非自己任期内的log entry是否成功提交，是不确定的
4. follower match并回退log优化
5. 为什么需要 preVote？
   假设没有preVote,网络隔离后的server 会有一个很大的term，网络恢复后，会导致其它节点变为follower开始选举，
   但日志不是最新的，不会成为新Leader。为避免网络分区节点重新加入集群，引起无效的Election。
