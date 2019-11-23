---
title: Zookeeper源码分析-概述
date: 2019-10-27 22:24:09
tags: [ZAB,zookeeper]
---
Zookeeper 是一个容错的分布式协调服务通常用于维护配置信息、服务发现、Leader选举等场景。采用ZAB(Zookeeper atomic broadcast)共识算法使得服务集群高可用，例如一个5节点的服务集群最多可以允许2个节点不可用(宕机、网络隔离)，集群仍能对外提供服务。
ZAB协议主要有四个阶段(ZabState)：ELECTION > DISCOVERY > SYNCHRONIZATION > BROADCAST。Zookeeper实际使用三阶段：Fast Leader Election > Recovery > Broadcast，集群节点有四种角色(ServerState)：
* LOOKING (初始角色)
* FOLLOWING（follower）
* LEADING (leader)
* OBSERVING (observer)

理解zookeeper的实现主可以顺着集群中节点角色及协议状态的转化过程，梳理出大致的。Zookeeper有单例和集群两种模式,本文基于3.5.5版本源码，从实现的角度介绍Zookeeper集群模式的程序入口、选举、崩溃恢复、广播等内容。

## ZK集群模式（QuorumPeerMain）
Zookeeper集群模式的入口类是QuorumPeerMain，主要完成两件事：配置解析及节点初始化并启动(QuorumPeer)。梳理配置解析对理解Zookeeper的整体实现有很大的帮助，比如Zookeeper使用了哪几类端口，作用分别是什么？
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

* 日志存储路径：dataDir、dataLogDir

### 节点启动

节点(QuorumPeer)启动主要完成三件事：
* 加载数据(loadDatabase)
* 监听客户端连接,客户端请求的路径：
  - ServerCnxnFactory 有两个实现类：NIOServerCnxnFactory、NIOServerCnxnFactory，
   处理网络连接(AcceptThread)、网络IO事件(SelectionThread)、读取连接数据（IOWorkRequest、NIOServerCnxn）
  - ZookeeperServer解析连接(processConnectRequest）和处理请求（processPacket）并submitRquest到处理链(Processor chain)

* 开始选举
  QuorumPeer里startLeaderElection方法，根据electionAlgorithm 配置创建选举实例,默认采用FastLeaderElection

## Leader选举
Zookeeper的Leader选举是基于ZAB算法，一种Quorum（多数派）算法，典型的Quorum算法，如Paxos、Raft。
高可用一致性服务的设计大多采用多副本来保证可用性。对于写请求，要在超过一般的节点写入成功才算成功,只要不超过半数的节点宕机都能保障集群可用。半数保证了集群中至少有一个节点拥有所有写入的最新数据，这保证了数据的可靠性。在Leader故障的情况下，Folllower会发起新的选举，可用节点中至少有一个节点保存了所有的数据，通过一定的比较，选举拥有全部数据的节点作为Leader可以使新Leader的数据同步简化和减少耗时。
ZAB 算法的详细解释不是本文的重点，但只有理解了ZAB算法，才能更好的阅读Zookeeper源码，算法请参考。FastLeaderElection类的入口是lookForLeader方法，QuorumPeer初始状态为LOOKING并开始选举。
选举流程主要包含如下步骤：
1. 向所有的Peer(包括自己)广播消息(ToSend),也即投票，投票就是将自己的Proposal广播，这也是***原子广播***的由来。Proposal包含如下内容:
       - 本节点sid
       - 本节点最大的Propse(提议)的id
       - 逻辑时钟（logicalclock），选举计数器
       - 节点选举状态(初始值LOOKING)
       - Propse的任期(Epoch),每个任期内的Leader对应唯一值
       - 验证配置QuorumVerifier

    每个Peer有两个收发消息的线程：WokerSender、WokerReceiver。其中Epoch非常重要,发生变化时会持久化到文件中，每次恢复从文件读取。
2. 收集选票，满足条件成为Leader或者Follower,没有足够的选票继续步骤1
   Peer根据收到的Notification的epoch和self.logicalclock的大小关系来更新Proposal:
      1. electionEpoch = self.logicalclock 则直接比较(sid,zxid,peerEpoch) 三元组
      2. electionEpoch < self.logicalclock 忽略Notification
      3. electionEpoch > self.logicalclock 更新本节点logicalclock,再比较(sid,zxid,peerEpoch) 三元组

   将收到的Notification加入选票统计，Peer Proposal一旦有更新，则广播自己的Proposal。判断选票是否达到了大多数，是则根据选举结果改变节点状态（LEADING or FOLLOWING），lookForLeader结束。


## 崩溃恢复（数据同步）
崩溃恢复主要关注Leader是如何处理各种异常情况下，数据的同步。数据同步的目的是为了保证副本状态的一致性，必须满足两个性质：
* 在任意副本上已提交的事务也必须在其余副本提交，通过SNAP和DIFF完成
* 没有提交的事务应当被废除，保证没有节点提交该事务，通过TRUNC来完成

必须处理两种异常：
* Leader在将事务已写入Commit log，未向Follower发起Proposal前宕机，恢复后废除该事务。
* Leader向Follower发起事务Proposal后宕机，新Leader需保证此事务正常commit。

关键字段:
- acceptedEpoch: the epoch number of the last NEWEPOCH message accepted
- currentEpoch: the epoch number of the last NEWLEADER message accepted
- lastProcessedZxid
- peerLastZxid

一旦某个Peer获得足够的选票，会变成LEADING状态，此时集群节点很快达成共识，其余节点变成FOLLOWING状态。
### Leader的lead方法
  * 重置tick,并加载zookeeper数据：ZKDataBase.loadDataBase
    - SnapShot
    - DataTree
    - DataNode
    - nodes
    - FileTxnSnapLog {
        txnlog(log.zxid),
        snaplog(snapshot.zxid),
        committedLog,
        save(lastProcessedZxid)
      }
    - Snapshot Thread
    - fastForwardFromEdits

  * 开启Leader连接处理线程，涉及类LearnerHandler处理follower请求/响应，方法syncFollower处理SNAP/DIFF/TRUNC。有四种情况：
    - peerLastZxid > maxCommitedLog: 直接TRUNC
    - minCommitedLog <peerLastZxid< maxCommitedLog: 利用内存中commited Proposals，DIFF同步或 TRUNC然后，DIFF同步
    - peerLastZxid < minCommitedLog: 利用事务日志加内存中commited Proposals，DIFF同步或 TRUNC然后，DIFF同步
    - 利用SNAP同步

### Follower的followerLeader方法

主要处理逻辑在三个方法中：
* syncWithLeader：崩溃恢复后与Leader同步数据
* read 和 process packet：原子广播阶段处理Leader事务proposal  

## 原子广播

事务请求采用Two-phased Commit，Zookeeper的写请求都会在Leader上以事务的方式提交，follower收到写请求会转发给Leader。
### LeaderZookeeperServer处理客户端的请求，采用一系列Processor：
  1. LeaderRequesetProcessor
  2. RrepRequestProcessor: 分配zxid
  3. ProposalRequestProcessor: 发起事务Proposal
     - SyncProcessorProcessor 写事务日志、批量提交事务日志、生成快照日志
       事务日志commit时机：
     - AckRequestProcessor：事务日志persistent后，mock leader vote the proposal

     等待follower ack 达到多数之后操作如下：
     * toBeApplied:将事务请求添加到toBeApplied队列（已达成大多数，但还未实际执行事务操作）
     * 发送Leader.COMMIT给所有的Followers
     * 并通知observer

  4. CommitProcessor
    * 非事务的request直接next processor
    * 事务操作，提交最早的事务
  5. ToBeAppliedRequestProcessor
  6. FinalRequestProcessor
    * processTxn: apply to dataTree
    * addCommittedProposal
    * 响应客户端的请求

### FollowerZookeeperServer处理客户端请求，采用一系列Processor:
  1. FollowerRequestProcessor
  2. CommitProcessor
  3. FinalRequestProcessor 专门用于处理Leader的事务日志请求
  4. SyncRequestProcessor
  5. SendAckRequestProcessor

## 客户端请求流程
未完，待续
## Q&A
1. Zookeeper 是如何区分未提交的事务呢？
   在Leader写入事务（zxid）日志，在向follower发起Proposal前宕机，正常的followers选出新leader。旧Leader节点恢复之后，发起数据同步，新Leader会发现不包含follower上的lastProcessZxid，Leader会向follower发送TRUNC。
