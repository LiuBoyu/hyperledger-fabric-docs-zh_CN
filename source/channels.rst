*此章节由 刘博宇 翻译，最后更新于2018.1.8* （`原文链接`_）

.. _`原文链接`: http://hyperledger-fabric.readthedocs.io/en/latest/channels.html

频道(Channel)
==============

一个 Hyperledger Fabric 的 **频道(Channel)** 是两个或多个特定网络成员之间相互通信的私有“子网”，用于进行隐私的和加密的交易(Transaction)。频道(Channel)包括如下的内容：会员(Member)（组织机构(Organization)），每个会员(Member)的锚节点(Anchor Peer)，共享的帐本(Ledger)，链码(Chaincode)程序和排序服务(Ordering Service)节点。网络中的每笔交易都在频道(Channel)中被执行，每个参与者都必须经过认证和授权才能在该通道上进行交易。每个加入频道(Channel)的节点(Peer)都拥有自己的身份标示，由会员服务提供商(Membership Services Provider - MSP)提供，它负责为其频道(Channel)中的节点(Peer)和服务(Service)，认证每一个节点(Peer)。

为了创建一个新的频道(Channel)，客户端SDK会调用配置系统(configuration system)的链码(Chaincode)，并引用相关的参数，例如 **锚节点(Anchor Peer)** 、 会员(Member)（组织机构(Organization)） 。之后，该请求会为频道(Channel)的账本(Ledger)创建一个创世区块(Genesis Block)，存储相关的配置信息，包括频道策略(channel policies)，会员(Member)和锚节点(Anchor Peer)。当新会员(Member)被添加到现有的频道(Channel)时，这个创世区块(Genesis Block)（又或者，是最近重新配置的配置区块(reconfiguration block)）将与新会员(Member)共享。

.. note:: 请参阅 :doc:`configtx` 章节，以更进一步了解有关配置交易(config transactions)的属性和数据结构。

对频道(Channel)中的每一个会员(Member)来说， **领导节点(Leading Peer)** 的选举将决定哪一个节点(Peer)会代表会员(Member)与排序服务(Ordering Service)进行通信。如果没有确定领导节点(Leading Peer)，则可以使用算法来确定领导节点(Leading Peer)。共识(Consensus)服务将交易排序，然后打包成一个区块(Block)，分发给每一个领导节点(Leading Peer)，最后，通过 **gossip** 协议，将其分发给其他的会员节点，及至整个频道(Channel)。

尽管一个锚节点(Anchor Peer)可以从属于多个频道(Channel)，并可以维护多个账本(Ledger)，但账本数据不可以从一个频道(Channel)传递到另一个频道(Channel)。以频道(Channel)为划分的账本(Ledger)是由如下因素定义和实现的，配置链码(configuration chaincode)，身份会员服务(identity membership service)和gossip数据传播协议。数据的传播（交易(transactions)、账本状态(ledger state)、频道(在线)会员(channel membership)）仅限于频道(Channel)中已验证会员资格的节点(Peer)。通过频道(Channel)，使节点(Peer)和账本数据(ledger data)相互隔离，对于那些需要交易具有隐私性和加密性的网络成员而言，在同一个区块链网络中，与商业竞争对手或其他受限制的成员共存，成为可能。

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
