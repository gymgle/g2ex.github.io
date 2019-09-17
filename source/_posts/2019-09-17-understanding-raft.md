title: "理解 Raft 分布式共识算法"
date: 2019-09-17 20:30:00
tag:
- raft
permalink: understanding-raft

---

## 0x00 简介

最近两年工作中对区块链技术接触较多，接下来可能要告一段落了。期间对 go-ethereum 进行过联盟链改造，使用 Raft 共识算法把以太坊的 TPS 提升到了 1K+。这里总结一下 Raft 算法，既是对自己经历对一种记录，也算是对他人对帮助。

Raft 算法是一个非常好理解（相比 Paxos 算法来说），也是一个非常受欢迎的共识算法，比如常用的服务发现、共享配置以及一致性保障的 etcd 和 Counsul 都使用了 Raft 算法来保证一致性。

## 0x01 什么是分布式共识算法

在分布式计算领域中有一个非常有名的 CAP 定理：一个分布式系统最多只能同时满足一致性（Consistency）、可用性（Availability）和分区容忍性（Partition tolerance）这三项中的两项。

一致性是指节点数据的一致，即所有节点在同一时间的数据完全一致。如果细分的话，一致性又可以分为强一致、弱一致和最终一致。比如我们经常使用的多副本关系型数据库满足的是强一致性，又因为要同时满足高可用，那么就弱化了分区容忍性（可以使用简单的网络拓扑来减少分区出错的可能）；公有的比特币、以太坊网络是属于最终一致，因为它们必须优先满足可用性和分区容忍性。

分布式网络中所有节点要想达成一致，就需要一个算法来促成这个一致，这个算法就是共识算法。我们常听说的 挖矿、PoS、DPoS、PBFT、Raft 都属于解决一致性的算法。

## 0x02 理解 Raft

Raft 中，节点通过心跳消息来保持通信，一个节点只会处于以下三种状态中的一种：
* Follower（跟从者）
* Candidate（候选人）
* Leader（领导者）

最开始时，所有的节点都是 follower，如果 follower 收不到 leader 的心跳消息，那么 follower 会变为 candidate，并向其他节点发起投票，如果该 candidate 节点收到了半数以上的选票（包括投给自己的一票），那么它就当选为新的 leader。这个过程被称为 **leader 选举**的过程。

接下来，leader 节点将带领所有节点对分布式网络中对数据更改达成一致，这个过程被称为**日志同步**。

日志同步的过程如下：
1. Leader 收到客户端到数据提交请求，leader 把请求作为一个条目（entry）加入到它到日志中，这个时候它不会立刻更新数据；
2. Leader 向所有的 followers 节点发送这个条目，这个发送的过程被称为 Append Entries；
3. Followers 节点收到 leader 的 Append Entries 请求后，向 leader 回复条目响应；
4. Leader 节点收集了半数以上的条目响应后，向客户端响应条目已确认，这时它才会更新自身节点的数据，同时向所有 fwllowers 节点发送条目确认的消息；

## 0x03 特殊情况

**Leader 选举过程中，如果没有收到半数以上的选票，该怎么办？**

Raft 中，有两种超时机制：选举超时和心跳超时。

每个 follower 会随机生成一个选举超时时间。任意节点当自身的选举超时时间结束后还没有收到 leader 的消息，那么它就重置这轮选举，变为 candidate 向其他节点发起新一轮投票。这样，最终总会选举出一个 leader 节点来。

**正常运行过程中，如果 leader 节点挂掉，会出现什么情况？**

这种情况下，会用到心跳超时机制。当 followers 在心跳超时后仍旧没有收到 leader 节点的心跳消息，那么 followers 节点就会发起新一轮投票，直到选举出新的 leader 来。

**网络分区的情况下会发生什么？**

网络故障导致导致节点被分隔到多个不连通的区域，在被隔离的区域中又会触发新的 leader 选举，对于隔离区域中包含半数以上的节点，选举就可能成功。当网络恢复后，followers 接受最大任期（term）和最新日志的 leader。这个思路蕾丝比特币、以太坊网络分叉后以最长区块为准的解决方案，确保了最终一致性。

## 0x04 Raft 参考资料

* 这里是一个不错的 Raft 算法演示网站，非常推荐用它演练一遍，增加对 Raft 的理解 http://thesecretlivesofdata.com/raft
* Raft 算法的 Github 主页，里边也包含了一个可视化演示 https://raft.github.io
* Raft 论文 https://raft.github.io/raft.pdf
* Raft 各种语言的实现 https://raft.github.io/#implementations
* 这里是对 Raft 算法对详解 https://zhuanlan.zhihu.com/p/32052223