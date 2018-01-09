*此章节由 刘博宇 翻译，最后更新于2018.1.8* （`原文链接`_）

.. _`原文链接`: http://hyperledger-fabric.readthedocs.io/en/latest/glossary.html

词汇表
========

术语非常重要，它让 Hyperledger Fabric 所有的用户和开发者就每个特定词语的含义达成一致。例如，链码是什么。文档将会根据需要引用词汇表，但如果你愿意的话，也可以一口气把词汇表通读一遍，这也是非常有启发性的！

.. _锚节点(Anchor Peer):

锚节点(Anchor Peer)
--------------------

一个节点(Peer)，它可以被频道(Channel)中的所有其他节点(Peer)发现和通讯。频道(Channel)中的每个 `会员(Member)`_ 都有一个锚节点(Anchor Peer)（或多个锚节点(Anchor Peer)，以防止单点故障），以允许从属于不同会员(Member)的节点(Peer)发现同频道(Channel)中的所有现有节点(Peer)。

.. _区块(Block):

区块(Block)
------------

一组有序的交易(Transaction)，以加密的方式链接到频道(Channel)中的前一个区块(Block)。

.. _链(Chain):

链(Chain)
----------

账本(Ledger)中的链(Chain)，是指一组交易日志，结构为哈希链接的交易区块(blocks of transactions)。节点(Peer)从排序服务(Ordering Service)接收交易区块(blocks of transactions)，并根据背书策略(Endorsement Policy)和并发冲突(concurrency violation)，标记区块(Block)中的交易(Transaction)是否有效，然后，将该区块(Block)追加到节点(Peer)文件系统中的哈希链(hash chain)上。

.. _链码(Chaincode):

链码(Chaincode)
----------------

链码(Chaincode)是一段运行在账本(Ledger)中的代码，通过对资产(Assets)和交易指令（业务逻辑）的编码来修改资产(Assets)。

.. _频道(Channel):

频道(Channel)
--------------

频道(Channel)是一块私有的区块链区域，实现了数据的隔离和保密。频道(Channel)所对应的账本(Ledger)在该频道(Channel)中，被所有的节点(Peer)共享，交易方必须通过该频道(Channel)的身份验证，才能与此账本(Ledger)交互。频道(Channel)由 `配置区块(Configuration Block)`_ 定义。

.. _提交(Commitment):

提交(Commitment)
-----------------

频道(Channel)中的每个 `节点(Peer)`_ 都会验证(validate)被排序后的交易区块(blocks of transactions)，然后，将区块(Block)提交（写入或追加）至该频道(Channel)中 `账本(Ledger)`_ 的各个副本。节点(Peer)还会标记每个区块(Block)中的每笔交易(Transaction)是否有效。

.. _并发控制版本检查(Concurrency Control Version Check):

并发控制版本检查(Concurrency Control Version Check)
----------------------------------------------------

并发控制版本检查(Concurrency Control Version Check)是频道(Channel)中各节点(Peer)间保持状态同步的一种方法。节点(Peer)以并行的方式执行交易，在提交至账本(Ledger)之前，节点(Peer)会检查在执行期间读取的数据是否有被修改过。如果读取的数据在执行和提交期间被改变过，就会引发并发控制版本检查(Concurrency Control Version Check)冲突，该交易就会在账本(Ledger)中被标记为无效，其值不会更新到状态数据库(State Database)中。

.. _配置区块(Configuration Block):

配置区块(Configuration Block)
------------------------------

系统链(System Chain)（排序服务(Ordering Service)）或频道(Channel)的配置数据，包含了会员和策略的配置。对频道(Channel)或整个网络的任何修改（例如，会员(Member)的离开或加入）都将会导致一个新的配置区块(Configuration Block)被追加到链中。这个区块(Block)将会包含创世区块(Genesis Block)的内容，以及其增量。

.. _共识(Consensus):

共识(Consensus)
----------------

共识(Consensus)是一个贯穿整个交易流程的广义术语，其用于表述，生成对排序(Order)达成的一致，以及确认构成区块(Block)的所有交易的正确性。

.. _当前状态(Current State):

当前状态(Current State)
------------------------

账本(Ledger)的当前状态(Current State)是指其链上的交易日志(transaction log)中所有键(Key)的最新值。节点(Peer)会将处理过的区块(Block)中的每笔有效交易的对应修改值，提交至账本(Ledger)的当前状态(Current State)。由于当前状态(Current State)代表了频道(Channel)中所有键(Key)的最新值，所以当前状态(Current State)也被称为世界状态(World State)。链码(Chaincode)执行交易提议(Transaction Proposal)就是针对的当前状态(Current State)。

.. _动态成员(Dynamic Membership):

动态成员(Dynamic Membership)
-----------------------------

Hyperledger Fabric支持动态添加/移除会员(Member)、节点(Peer)和排序服务(Ordering Service)节点，而不会影响整个网络的可操作性。当业务关系调整或因为各种原因而需要添加/移除实体时，动态成员(Dynamic Membership)机制至关重要。

.. _背书(Endorsement):

背书(Endorsement)
------------------

背书(Endorsement)是指特定节点(Peer)执行链码(Chaincode)交易并向客户端程序返回提议响应(Proposal Response)的过程。提议响应(Proposal Response)包括链码(Chaincode)执行的响应消息，结果（读取集合和写入集合）和事件，以及作为链码(Chaincode)在节点(Peer)中执行证据的签名。链码(Chaincode)程序具有相应的背书政策(Endorsement Policy)，其中指定了背书节点(Endorsing Peer)。

.. _背书策略(Endorsement Policy):

背书策略(Endorsement Policy)
-----------------------------

定义了频道(Channel)中执行交易的节点(Peer)（其必须以指定链码(Chaincode)程序执行），以及响应结果（背书(Endorsement)）的必要组合条件。背书策略(Endorsement Policy)可以指定交易(Transaction)的背书方式，以背书节点(Endorsing Peer)的最少数量、或以背书节点(Endorsing Peer)的最少百分比，又或者，分配了某一特定链码(Chaincode)程序的所有背书节点(Endorsing Peer)。背书策略(Endorsement Policy)由背书节点(Endorsing Peer)基于应用程序和对抵御不良行为的期望水平来组织管理的。交易(Transaction)必须满足背书政策(Endorsement Policy)，然后才能被提交节点(committing peers)标记为有效。安装(Install)和实例化(Instantiate)交易也需要明确的背书策略(Endorsement Policy)。

.. _Hyperledger-Fabric-CA:

Hyperledger Fabric CA
---------------------

Hyperledger Fabric CA是默认的CA(Certificate Authority)组件，它向网络会员及其用户颁发基于PKI的证书。CA为每个会员颁发一个根证书（rootCert），为每个授权用户颁发一个注册证书（eCert）。

.. _创世区块(Genesis Block):

创世区块(Genesis Block)
------------------------

初始化区块链网络或频道(Channel)的配置区块(Configuration Block)，也是链上的第一个区块(Block)。

.. _Gossip协议:

Gossip协议
------------

Gossip数据传播协议有三个功能：

1. 管理节点发现(peer discovery)和频道(在线)会员(channel membership)；
2. 在频道(Channel)中的所有节点(Peer)间传播账本数据；
3. 在频道(Channel)中的所有节点(Peer)间同步账本状态；

请参阅 :doc:`Gossip <gossip>` 主题了解更多详情。

.. _初始化(Initialize):

初始化(Initialize)
-------------------

初始化链码(Chaincode)程序的方法。

安装(Install)
--------------

将链码(Chaincode)部署到节点(Peer)文件系统上的过程。

实例化(Instantiate)
--------------------

启动和初始化特定频道(Channel)上链码(Chaincode)程序的过程。在实例化(Instantiate)之后，安装链码(Chaincode)的节点(Peer)就可以接受链码调用了。

.. _调用(Invoke):

调用(Invoke)
-------------

用于调用链码(Chaincode)内的函数。客户端程序通过向节点(Peer)发送交易提议(Transaction Proposal)来调用链码(Chaincode)。节点(Peer)会执行链码(Chaincode)并将经过背书(Endorsement)的提议响应(Proposal Response)返回给客户端程序。客户端程序收集到满足背书策略(Endorsement Policy)所需的提议响应(Proposal Response)后，将提交交易结果，然后，排序(Ordering)，验证（Validation）和提交(Commitment)。客户端程序可以选择不提交交易结果。例如，如果只是查询账本(Ledger)，客户端程序通常不会提交只读交易，除非希望出于审计的目的，将读取帐本(Ledger)的记录写入日志。调用(Invoke)包括一个频道ID(Channel ID)，要调用的链码(Chaincode)函数和一个参数数组。

.. _领导节点(Leading Peer):

领导节点(Leading Peer)
-----------------------

每个 `会员(Member)`_ 在其订阅的每个频道上都可以拥有多个节点(Peer)。这些节点(Peer)中的其中一个会作为频道(Channel)中的领导节点(Leading Peer)，代表会员(Peer)与网络排序服务(Ordering Service)进行通信。排序服务(Ordering Service)会首先将区块(Block)分发给频道(Channel)中的领导节点(Leading Peer)，然后领导节点(Leading Peer)再会将这些区块(Block)分发给同一会员(Member)下的其他节点(Peer)。

.. _账本(Ledger):

账本(Ledger)
-------------

账本(Ledger)是指一个频道(Channel)所对应的链和其当前的状态数据(State Data)，由频道(Channel)上的每个节点(Peer)共同维护。

.. _会员(Member):

会员(Member)
-------------

拥有网络唯一根证书的合法独立实体。像节点(Peer)和应用程序客户端这样的网络组件将会链接到会员(Member)。

.. _MSP:

会员服务提供商(Membership Service Provider - MSP)
---------------------------------------------------

会员服务提供商(Membership Service Provider - MSP)是一个系统抽象组件，它向客户端和节点(Peer)提供证书，以便它们参与 Hyperledger Fabric 的网络。客户端使用证书，对其交易进行认证(authenticate)，节点(Peer)使用证书来认证(authenticate)交易处理结果（背书(Endorsement)）。尽管该接口与系统的交易处理组件密切相关，但它可以可插拔的形式，平滑地替换已定义的会员服务(Membership Services)组件，而不会修改系统的交易处理组件的核心。

.. _会员服务(Membership Services):

会员服务(Membership Services)
------------------------------

会员服务(Membership Services)在许可的区块链网络上认证、授权和管理身份。在节点(Peer)和排序节点(Orderer)中运行的会员服务(Membership Services)代码都会认证和授权区块链的操作。它是基于PKI的会员服务提供商(Membership Service Provider - MSP)的抽象实现。

.. _排序服务(Ordering Service):

排序服务(Ordering Service)
---------------------------

一组节点集合，负责将交易(Transaction)排序打包进一个区块(Block)中。排序服务(Ordering Service)独立于节点(Peer)流程存在，并以先到先得的原则为网络上所有频道(Channel)的交易进行排序。排序服务(Ordering Service)被设计为支持可插拔的实现方式，目前，已经实现了 SOLO 和 Kafka 。排序服务(Ordering Service)是整个网络的通用绑定，它包含了与每个 `会员(Member)`_ 关联的加密身份资料。

.. _节点(Peer):

节点(Peer)
-----------

一个网络实体，负责维护账本(Ledger)并运行链码(Chaincode)容器，以对账本(Ledger)执行读写操作。节点(Peer)由会员(Member)拥有和维护。

.. _策略(Policy):

策略(Policy)
-------------

有背书(Endorsement)策略，验证(Validation)策略，链码管理(chaincode management)策略和网络/频道管理(network/channel management)策略。

.. _提议(Proposal):

提议(Proposal)
---------------

针对频道(Channel)中某一节点(Peer)的背书请求(a request for endorsement)。每个提议(Proposal)或者是实例化(Instantiate)请求，或者是调用(Invoke)（读取/写入）请求。

.. _查询(Query):

查询(Query)
------------

查询(Query)是一个链码调用(chaincode invocation)，它读取帐本(Ledger)的当前状态(Current State)，但不写入帐本(Ledger)。链码(Chaincode)函数可以查询帐本(Ledger)中的某些键，或者查询帐本(Ledger)中的一组键。由于查询(Query)不会改变帐本(Ledger)的状态，因此客户端应用程序通常不会提交这些只读交易进行排序(Ordering)，验证(Validation)和提交(Commit)。虽然并不常见，但客户端应用程序也可以选择提交只读交易进行排序(Ordering)，验证(Validation)和提交(Commit)，例如，如果客户希望得到账本(Ledger)链中的审计证据，它记录了某个具体时间点上读取了某个特定的账本状态(ledger state)。

.. _SDK:

SDK
---

Hyperledger Fabric客户端SDK为开发人员提供了一个结构化的库环境，用于编写和测试链码(Chaincode)应用程序。SDK完全可以通过标准接口实现配置和扩展，像签名的加密算法、日志框架和状态存储(state store)这样的组件都可以轻松地实现替换。SDK提供了用于交易处理(transaction processing)，会员服务(membership service)，节点遍历(node traversal)和事件处理(event handling)的API。SDK将会有多种版本：Node.js、Java和Python。

.. _状态数据库(State Database):

状态数据库(State Database)
--------------------------

为了从链码(Chaincode)中高效地读写和查询，当前状态(Current State)数据被存储在状态数据库(State Database)中，支持的数据库包括levelDB和couchDB。

.. _系统链(System Chain):

系统链(System Chain)
---------------------

系统链(System Chain)包含了在系统级别定义网络的配置区块(Configuration Block)。系统链(System Chain)存在于排序服务(Ordering Service)中，与频道(Channel)类似，具有包含以下信息的初始配置：MSP信息、策略和配置详情。对整个网络的任何变化（例如新的组织机构(Org)加入或者添加新的排序节点）将导致新的配置区块(Configuration Block)被添加到系统链(System Chain)中。

系统链(System Chain)可看做是一个频道(Channel)或一组频道(Channel)的公用绑定(common binding)。例如，一系列金融机构可以组成一个财团（通过系统链(System Chain)表示），然后根据其各自不同的业务议程，来创建相对应的频道(Channel)。

.. _交易(Transaction):

交易(Transaction)
------------------

调用(Invoke)或实例化(Instantiate)用于排序(Ordering)，验证(Validation)和提交(Commitment)的结果。调用(Invoke)是用来请求从账本(Ledger)中读取和写入数据。实例化(Instantiate)是用来请求启动和初始化频道(Channel)中的链码(Chaincode)。应用程序客户端收集来自背书节点(Endorsing Peer)调用(Invoke)或实例化(Instantiate)的响应，并将结果和背书(Endorsement)打包进用于排序(Ordering)，验证(Validation)和提交(Commitment)的交易中。

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
