---
layout: post
title: 分布式一致性算法
categories: [Algorithm, Distributed System]
description: 分布式一致性算法 Paxos, Raft, Gossip 比较
keywords: Algorithm, Paxos, Raft, Gossip, CAP, BASE
---

## 分布式理论

分布式一致性领域，最重要的两个理论是 CAP 和 BASE。

CAP 理论说的是衡量分布式系统有三个维度：一致性（Consistency）、可用性（Availablity）和分区容忍性（Partition Tolerance）。

CAP 理论重要之处是指出了一个分布式系统只能满足三项中的两项而不可能满足全部三项。

网络发生分区时，如果要保证一致性，那么就需要把消息复制到所有节点，显然由于网络分区复制会失败，系统就处于不可用状态，即无法保证可用性；如果保证可用性，那么可以忽略复制失败，各分区继续保持可用，那么需要牺牲一致性；如果无法容忍网络分区，那么可满足一致性和可用性，但分区容忍性不能满足。

CAP 理论无法全部满足，因此出现了 BASE 理论，基本思想是“即使无法做到强一致性，但每个应用都可以根据自身的业务特点，采用适当的方式来使系统达到最终一致性”，BASE 理论通过牺牲强一致性来获得可用性，允许数据短时间内不一致，但最终状态需要一致。

常见的分布式一致性算法包括 Paxos、Raft、GOSSIP 等。

## Paxos 算法

Paxos 算法解决的问题是在分布式系统中，如何使不同节点间对一个 key 的取值达成共识（即 key 更新为同一个值）。

Basic Paxos 只能对一个值形成决议，一般只用于理论研究， Multi-Paxos 可以用于连续确定多个值。

节点划分为三种角色：

- 提案者（Proposer）：对 key 的值提出自己的值
- 决议者（Acceptor）：对提案者提案进行投票，是否选择这个提案，生成最终决策
- 学习者（Learner）：学习达成共识的决策，提案过程学习者不参与

一般集群中的所有节点都同时具有上述三个角色。提案者的提案是由 Client 客户端发起的，提案者可以认为是 Client 客户端的代理。

### Basic Paxos

目标：一次 Paxos 共识对 key 的一个取值形成决议，且最终只决议出一个值。

过程推导：

1. 当集群系统中超过半数的 Acceptor 接受了一个议案，那么这个议案就可以作为最终决策

2. 按照先来先服务原则，Acceptor 接受最先收到的议案，拒绝后续收到的议案，存在问题是多个 Proposer 同时发出议案，导致没有一个议案能得到超过半数的选票，Acceptor 必须能够接受多个议案

3. Acceptor 能够接受多个议案，当某个议案被超过一半的 Acceptor 接受后，这个议案就被选定了。为了区别 Acceptor 接受的多个议案，要求 Proposer 提出的每个议案携带一个编号，议案变为二元组（Num, Value），所有 Proposer 发出的议案编号按发出的先后顺序递增，可以用时间戳+每个节点的编号表示议案编号

4. Acceptor 如果能接受多个议案，有可能会出现多个不同值都被最终选中的情况，解决办法是按先来先服务的原则，Acceptor 接受最先收到的议案(n, v)，后续拒绝编号小于等于 n 的议案，接受编号大于 n 的议案，且要求编号大于 n 的议案的值也必须为 v ，这样既可以接受后来的提案，又可以保证只接收一个值。

5. 可以看出 Proposer 的议案是否被接受，首先取决于议案的编号，其次取决于 Acceptor 是否接受过其他 Proposer 的议案，因此将提案过程分为两个阶段，阶段一准备（Prepare）提案，阶段二接受（Accept）提案，决议形成后需要让所有节点学习到该决议（学习阶段已经形成了决议，故不属于提案过程）。

    1. 阶段一（Prepare）：Proposer 生成全局唯一且递增的提案编号，向所有 Acceptors 发送 Prepare 请求，请求只携带提案编号，无需提案值。Acceptors 收到 Prepare 请求后，承诺不再接受编号**小于等于**当前请求的 Prepare 请求，且承诺不再接受编号**小于**当前请求的 Accept 请求。在此基础上更新收到过的最大请求编号，回复 Proposer 已经 Accept 过的提案中编号最大的那个提案的编号和值，如果没有则返回空值。（Q：如果 Perpare 请求的编号小于收到过的请求编号，发拒绝 Proposer 报文？）
    2. 阶段二（Accept）：Proposer 收到多数 Acceptors 的应答后，从应答中选择编号最大的提案的值，作为本次要发起的提案的值，如果所有应答的提案值都为空，则可以自己决定一个提案值。然后携带当前提案编号，向所有 Acceptors 发送 Accept 提案请求。（Q：如果没有收到半数以上应答，重新进行阶段一？）。Acceptor 收到 Accept 请求后，在不违背之前的承诺下，拒绝或者接受并持久化当前提案编号和提案值，即如果当前 Accept 提案编号小于之前接受过的 Prepare 或者 Accept 请求编号，则拒绝当前提案，并返回之前的提案编号；否则接受并持久化当前提案编号和提案值。Proposer 接收到过半数请求后，如果发现有返回值大于本次提案编号，表示有更新的提议，跳转到1；否则 value 达成一致。
    3. 第三阶段（Learn）：Proposer 收到多数 Acceptors 的 Accept 后，决议形成，将形成的决议发送给所有 Learners 。

6. 活锁（Livelock）问题：如果 Acceptor 在接受 Proposer-A 的 Prepare 和 Accept 之间有 Proposer-B 提出编号更大的 Prepare 请求，那么 Proposer-A 将会 Accept 失败，重新用编号更大的提案进行 Prepare 请求，两个 Proposers 交替 Prepare 成功，而 Accept 失败，形成活锁。活锁建议的解决方案是在 Proposers 中选举一个 Leader 负责提案（即 Multi-Paxos 方案）。另外活锁是由于两个 Proposers 步调交替执行了，可以在 Accept 阶段失败时，增加一段随机的等待时间，再新获取提案编号重新执行。

7. 可以看出 Basic Paxos 的第一阶段主要确定最大编号和值，第二阶段接受提案。可能发生的情况有：

    - Prepare 阶段最先发出的提案就是 Acceptor 收到的最大编号，且 Accept 阶段选定该提案

    - Prepare 阶段的提案编号较大，但 Acceptor 在此之前已经确认过其他编号较小的 Accept 请求值 V ，这种情况 Proposer 将更新自己的值为 V ，并成功进行 Accept 提交

    - Prepare 阶段的提案编号较小，Acceptor 在此之前收到更大编号的 Prepare 提案，此时 Proposer 将被拒绝，重新开始 Prepare

    - 极端情况两个 Proposers 交替 Prepare 成功，而 Accept 失败，形成活锁

    - Accept 阶段半数以上回复同意提案，则提案达成共识

    - Accept 阶段有返回值大于本次提案编号，表示存在更新的提议，重新开始提案

    - 超时：Prepare 阶段或 Accept 阶段 Proposer 超过一定时间没有收到任何拒绝或半数以上同意回复，那么本次超时，需要重新提案

Basic Paxos 伪代码如下图：

![Basic Paxos 伪代码](/images/posts/distributed-system/basic-paxos-pseudocode.jpg)

Basic Paxos 缺点：

- 只能对一个值形成决议，无法对连续值进行决议
- 决议过程需要至少两次网络来回，高并发情况可能需要更多次数的网络来回
- 极端情况形成活锁

### Multi-Paxos

Basic Paxos 的上述缺点造成其几乎只用来做理论研究，业界真正生产应用的是基于原始 Paxos 算法改进的 Multi-Paxos 算法，该算法主要改进：

1. 在所有 Proposers 中选举一个 Leader ，由 Leader 唯一提交 Proposal 给 Acceptors 进行表决，这样没有 Proposer 竞争，解决了活锁问题。在系统中仅有一个 Leader 进行值提交的情况下，可以跳过 Prepare 阶段，从而将两阶段变成一阶段，提高效率。

2. Multi-Paxos 允许有多个自认为是 Leader 的节点并发提交 Proposal 而不影响其安全性，这样的场景即退化为 Basic Paxos。为了区分连续提交的多个 Paxos 实例，每个实例使用一个 Instance ID 标识，Instance ID 由 Leader 本地递增生成即可。

另外， Leader 的选举也是一次决议的形成，可以通过执行一次 Basic Paxos 实例选举出 Leader 。在 Leader 宕机后服务临时不可用，需要重新选举 Leader 继续服务。

Multi-Paxos 通过改变 Prepare 阶段的作用范围至后面 Leader 的连续提交，从而使得 Leader 的连续提交只需要执行一次 Prepare 阶段，后续只需要执行 Accept 阶段，将两阶段变为一阶段，提高了效率。

## Raft 算法

Raft 将节点划分为三类：

- Leader：接受客户端请求，并向 Follower 同步请求日志，当日志同步到大部分节点后，通知 Follower 提交日志
- Follower：接受并持久化 Leader 同步日志，在 Leader 告知日志可以提交后，提交日志
- Candidate：Leader 选举过程中的临时角色

系统在任意时刻最多只有一个 Leader，正常工作期间只有 Leader 和 Followers。

 Raft 算法从多副本状态机的角度提出，用于管理多副本状态机的日志复制。采用分治的思想，将分布式系统一致性问题划分为多个子问题：

- Leader 选举（Leader election）
- 日志同步（Log replication）
- 安全性（Safety）
- 日志压缩（Log compaction）
- 成员变更（Membership change）

### Leader 选举

- 初始启动时节点作为 Follower
- Leader 通过心跳维持任期内统治，每个任期有一个递增的 term 编号进行标记
- Follower 在选举超时时间内没有收到 Leader 心跳，就会等待一段随机时间后发起一次 Leader 选举
- 将当前 term 加一，转换为 Candidate，给自己投票并给集群中其它服务器发送 RequestVote RPC
- 需保证选举出的 Leader 上一定具有最新的已提交的日志，即 Candidate 的投票报文中需要包含最新提交日志信息
- term 作为轮次记录作用，Follower 在一轮投票中只能投一票，即收到的 term 比当前记录的大，则投一票，并更新 term，否则投拒绝票

Follower 发起的 Leader 选举结果有以下三种情况：

- 赢得了多数的选票，成功选举为 Leader
- 收到了 Leader 的消息，表示有其它服务器已经抢先当选了 Leader
- 没有服务器赢得多数的选票， Leader 选举失败，等待选举时间超时后发起下一次选举

### 日志同步

- Leader 接收客户端请求（工程上一般采用 Follower 转发客户端请求）
- Leader 把请求作为日志条目（log entries）加入自己的日志，并行向 Follower 发起 AppendEntries RPC 复制日志条目
- 一条日志被复制到大多数服务器上，Leader 将这条日志应用到自身状态机上，并返回客户端执行结果
- Leader 通知其他节点提交日志（即通知其他节点将日志应用到自己的状态机上）
- 强一致性与弱一致性：如果 Leader 在确认半数以上 Follower 都提交了日志后，再返回客户端执行结果，那么可以认为是强一致性的（比如 etcd）；如果 Leader 更新自身后即返回客户端结果，同时异步发送 Follower 更新日志，那么可以认为是弱一致性的（redis 即是异步写 aof 和更新副本的）

## GOSSIP 算法

GOSSIP 算法是一种 P2P 算法，去中心化，没有主备之分，多个参与者之间能够达到最终一致性。存在问题是参与者太多可能造成参与者之间的网络交互信息量较大，系统达到最终一致所用时间较长。

Redis 集群采用 GOSSIP 算法在多个参与节点之间交互集群和分片信息。

## 参考

- [Paxos算法详解](https://zhuanlan.zhihu.com/p/31780743)
- [Raft算法详解](https://zhuanlan.zhihu.com/p/32052223)
- [从 Paxos 到 Raft，分布式一致性算法解析](https://www.infoq.cn/article/us5GJQQZ8bMbEHa25Io0)