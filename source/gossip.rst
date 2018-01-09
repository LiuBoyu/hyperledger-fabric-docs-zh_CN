*此章节由 刘博宇 翻译，最后更新于2018.1.5* （`原文链接`_）

.. _`原文链接`: http://hyperledger-fabric.readthedocs.io/en/latest/gossip.html

Gossip数据传播协议
====================

*译者注: gossip协议是一个神奇的协议。它常用于P2P的通信协议，这个协议就是模拟人类中传播谣言的行为而来。简单的描述下这个协议，首先要传播谣言就要有种子节点。种子节点每秒都会随机向其他节点发送自己所拥有的节点列表，以及需要传播的消息。任何新加入的节点，就在这种传播方式下很快地被全网所知道。这个协议的神奇就在于它从设计开始就没想到信息一定要传递给所有的节点，但是随着时间的增长，在最终的某一时刻，全网会得到相同的信息。*

Hyperledger Fabric通过拆分工作量为交易执行节点（背书(Endorsing)和提交(Committing)）和交易排序节点，来优化区块链网络的性能，安全性和可伸缩性。这种网络操作的解耦需要一个安全的、可靠的和可伸缩的数据传播协议，以确保数据的完整性和一致性。为了满足这些要求，Hyperledger Fabric实现 **gossip数据传播协议** 。

Gossip协议
------------

节点(Peer)利用gossip以可扩展的方式广播账本(Ledger)和频道(Channel)的数据。Gossip的消息发送是连续的，频道(Channel)上的每个节点(Peer)都不断地从多个其他节点(Peer)接收最新的和一致的账本数据。每个gossip消息都会被签名，从而，拜占庭参与者(Byzantine Participants)发送的伪造消息将会很容易地被识别出来，并且，阻止这些消息被分发到不想要的目标。节点(Peer)会受到网络延时、网络分区或其他因素的影响，从而导致丢失区块(Block)，但最终会通过联系那些拥有缺失区块的节点(Peer)，而同步到最新的帐本状态。

基于gossip的数据传播协议在 Hyperledger Fabric 网络中，主要执行三项功能：

1. 通过持续不断地识别可用的会员节点，并最终检测已离线的节点，来管理节点发现(peer discovery)和频道(在线)会员(channel membership)。
2. 在频道(Channel)中的所有节点(Peer)之间传播账本数据。可以识别与频道(Channel)中其余节点不同步的节点(Peer)，标示出丢失的数据块，并通过复制正确的数据来同步自身。
3. 通过以P2P的方式更新账本数据，使新连接的节点(Peer)加速。

Gossip的广播，首先，是由节点(Peer)接收频道(Channel)上的其他节点(Peer)消息，然后，将这些消息转发给频道(Channel)中的多个随机节点(Peer)，这个数量是一个可配置的常量。节点(Peer)也可以使用拉取(pull)机制，而不是等待信息的传递。这个循环不断地重复，最终，频道(在线)会员(channel membership)，账本(Ledger)和状态信息，将会持续不断地被同步至最新的结果。为了传播新的区块，该频道(Channel)中的 **领导** 节点(leader peer)将从排序服务(Ordering Service)中拉取(pull)数据，并开始向其他节点(Peer)传播gossip消息。

Gossip消息发送
---------------

在线节点(Peer)通过持续不断地广播“活着”的消息，来表明他们的可用性，每个消息都包含了 **公钥基础设施(Public Key Infrastructure - PKI)** ID和发件人针对消息的签名。节点(Peer)通过收集这些“活着”的消息来维护频道(在线)会员(channel membership)。如果没有节点(Peer)收到来自特定节点(Peer)的“活着”消息，则这个“死了”的节点将最终从频道(在线)会员(channel membership)中清除。因为“活着”消息是被加密签名的，并且，由于缺乏根证书授权机构（CA）的签名密钥，所以，恶意节点(Peer)永远不能冒充其他节点(Peer)。

除了自动转发收到的消息之外，频道(Channel)中的节点(Peer)之间，还会有一个状态核对过程，来同步 **世界状态(world state)** 。每个节点(Peer)持续不断地从该频道(Channel)的其他节点(Peer)中拉取(pull)区块，以便在发现差异的情况下修复自己的状态(state)。基于gossip的数据传播不需要保持固定的网络连接，因此，此过程非常可靠地保证了共享帐本的数据一致性和完整性，包括对节点崩溃的容错性。

由于频道(Channel)是相互隔离的，所以一个频道(Channel)上的节点(Peer)不能发送消息或分享信息给其他任何的频道(Channel)。虽然任意节点(Peer)可以从属于多个频道(Channel)，但通过应用基于节点频道订阅(peers' channel subscription)的消息路由策略(message routing policy)，分区的消息发送机制(partitioned messaging)可以防止区块被传播到不在此频道(Channel)中的节点(Peer)。

**备注:**

1. P2P消息的安全性由节点(Peer)的TLS层处理，不需要签名。节点(Peer)由CA分配的证书进行认证。尽管也使用TLS证书，但在gossip层中，节点(Peer)证书依然会被验证。账本区块(ledger blocks)由排序服务(Ordering Service)进行签名，然后交付给频道(Channel)中的领导节点(leader peers)。

2. 身份验证由节点(Peer)的MSP(Membership Service Provider)负责管理。当节点(Peer)首次连接到该频道(Channel)时，TLS会话将与会员ID(membership identity)相绑定。实质上，每一个节点(Peer)都对连接节点(connecting peer)进行了身份验证，以确保其在当前网络和频道(Channel)中的会员资格。

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
