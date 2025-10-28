---
title: raft（未完）
date: 2016-04-26
tags: 
  - 一致性协议

slug: raft-protocol
---

#### 5.1 基本概念

​ 一个Raft集群包括几个节点。五是一个典型的数量，这个数量使系统能够容错两次。每个节点在任何时候都处于以下三种状态之一：leader,follower,或candidate。运行正常的情况下，集群中有一个leader，其他节点都是followers。follower是被动的：它们不主动发起请求，只会响应来自leader或candidate的请求。leader处理所有的客户端请求（如果一个客户端与follower通讯，那么follower将转发请求给leader）。第三种状态，candidate，用于选举leader，选举将在5.2节详细说明。图例4展示了三种状态以及它们之间的转换，接下来就讨论状态之间的转换。

​ Raft被分割为不定长的terms。terms用连续的整数来编号。每个term从选举开始，选举中candidates将试图成为leader。如果一个candidate赢得了选举，那么该term剩下的时间中它将作为leader存在。某些情形下，选举会以分票告终。这种情况下，该term没有leader，一个新的term会马上开始。Raft确保每个term最多只有一个leader。

​ 不同的节点可能在不同的时间点察觉到term的转换，在某些情况下，一个节点不会察觉到某次选举甚至整个term。term在Raft中起到扮演逻辑计时器的角色，它使各个节点能检测到过时的信息，比如过期的leader。每个节点都会存储current term，这个值会随着时间递增。节点之间通讯时会交换current terms信息，如果一个节点的current term小于其他人的，它会把它更新为较大的那个值。如果一个candidate或者leader发现它的term已经过时，它将立刻转为follower。如果一个节点接收到的请求包含了过时的term，它会拒绝该请求。

​ Raft节点之间通过远程过程调用（RPCs）通讯，最基本的一致性算法只需要两种类型的RPCs。RequestVote RPCs 在选举期间由candidates发起，Append-Entries RPCs 由leader发起，用于复制log，或者模拟心跳。第7节增加了第三种PRCs用于节点之间交换快照。如果没有收到响应，节点会及时的重试RPCs。为了更好的性能，RPCs将并行的进行。

#### 5.2选举

​ Raft使用心跳机制来触发选举。节点启动之后，初始状态是followers。只要节点持续的收到来自leader或candidate的合法RPCs，它会一直保持follower的状态。Leaders周期性的发送心跳（不带leg entries的AppendEntries RPCs）来维持状态。如果一个follower一段时间没有收到请求，那么会发生选举超时，节点假定现在没有可用的leader，并开始选举一个新的leader。

​ 当选举开始时，follower将term加一并转为candidate状态。然后它为自己投票并并行的向集群中的节点发起RequestVote RPCs。candidate保持其状态直到以下三种情况之一发生：(a)赢得选举，(b)其他节点声明自己是leader，(c)一段时间过去了而没有人胜出。以上三种结果会在下面的篇幅中分别讨论。

​ 如果candidate收到集群中大部分节点相同term的投票，那么它将赢得选举。一个节点在一个term中最多只投一次票，按照先到先得的原则。多数票胜出原则保证了一个term当中最多只有一个candidate能赢得选举。一旦candidate赢得了选举，它的状态将转为leader。接着它会向其他节点发送心跳来防止新的选举。

​ 等待投票的过程中，candidate可能会受到其他节点的AppendEntries RPC表示自己是leader。
