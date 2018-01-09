*此章节由 刘博宇 翻译，最后更新于2018.1.4* （`原文链接`_）

.. _`原文链接`: http://hyperledger-fabric.readthedocs.io/en/latest/txflow.html

交易流程
=========

本文档概述了标准资产(Standard Asset)交换过程中发生的交易机制。该场景包括两个客户，A和B，分别是买萝卜和卖萝卜。他们在网络中各有一个节点(Peer)，通过网络他们发送交易(Transactions)并与账本(Ledger)进行交互。

.. image:: images/step0.png

**前提假设**

这个流程假设一个频道(Channel)已经建立并运行。应用程序用户已经注册，并且也完成了组织机构在证书颁发机构（CA）的注册，并已收到了所需要的加密材料，用以在网络上进行身份验证。

链码(Chaincode)（包含了代表萝卜市场初始状态的一组键值对）已经安装在节点(Peer)之上并且也已经在频道(Channel)上实例化了。链码(Chaincode)内包含了一组交易指令和商定萝卜价格的业务逻辑。背书策略(Endorsement Policy)也已经为此链码(Chaincode)设置完成，并规定，任意一笔交易，都必须由节点(Peer)A和节点(Peer)B的共同背书(Endorse)。

1. **客户A发起交易**

.. image:: images/step1.png

都发生了什么呢？ - 客户A正在发送购买萝卜的请求(Request)。该请求(Request)的发送目标是分别代表客户A和客户B的节点(Peer)A和节点(Peer)B。背书策略(Endorsement Policy)规定了任意一笔交易都必须由两个节点(Peer)共同背书(Endorse)，因此请求(Request)被同时发送到节点(Peer)A和节点(Peer)B。

接下来，我们要创建交易提议(Transaction Proposal)。应用程序利用SDK(Node,Java,Python)的API生成交易提议(Transaction Proposal)。交易提议(Transaction Proposal)是一个请求(Request)，以调用链码(Chaincode)函数，因此，数据可以被读取或写入账本(Ledger)（例如：为资产(Assets)写入新的键值对）。SDK将交易提议(Transaction Proposal)打包成框架所需要的正确格式(Protocol Buffers - gRPC)，并以用户的加密凭证(Cryptographic Credentials)为此交易提议(Transaction Proposal)生成唯一签名。

2. **背书节点(Endorsing Peers)校验签名并执行交易**

.. image:: images/step2.png

背书节点(Endorsing Peer)将会对交易提议(Transaction Proposal)进行如下的校验:

* （1）格式是否正确
* （2）以前是否被提交过（重放攻击保护）
* （3）签名是否有效（使用MSP）
* （4）提交者（例如，客户端A）是否在该频道(Channel)上被授权执行提议操作（即，每个背书节点(Endorsing Peer)都要确保提交者满足频道(Channel)的写入策略(Writers Policy)）。

背书节点(Endorsing Peer)将交易提议(Transaction Proposal)作为被调用的链码(Chaincode)函数的输入参数。然后，链码(Chaincode)执行于当前的状态数据库(State Database)之上，以生成交易结果，包括响应值，读取集合和写入集合。此时并没有更新账本(Ledger)。这些值的集合以及背书节点(Endorsing Peer)的签名将作为“交易提议响应(Transaction Proposal Response)”回传给SDK，该SDK将负责解析这些数据，以供应用程序使用。

*{MSP是一个节点组件(Peer Component)，用来校验来自客户端的交易请求并为交易结果签名（背书）。写入策略(Writers Policy)在频道(Channel)创建时被定义，并确定哪个用户有权限向该频道(Channel)提交交易。}*

3. **检查交易提议响应(Transaction Proposal Response)**

.. image:: images/step3.png

应用程序会验证所有背书节点(Endorsing Peer)的签名并比较所有的交易提议响应(Transaction Proposal Response)，以确定所有的交易提议响应(Transaction Proposal Response)是否相同。如果链码(Chaincode)仅仅是查询账本(Ledger)，则应用程序将会检查查询响应(query response)，并且通常不会将此交易(Transaction)提交给排序服务(Ordering Service)。如果客户端应用程序打算将交易(Transaction)提交给排序服务(Ordering Service)以更新账本(Ledger)，则应用程序在提交之前，将会确定被指定的背书策略(Endorsement Policy)是否已经得到满足（即，节点(Peer)A和节点(Peer)B都已背书）。从架构的层面来说，即使应用程序选择不检查交易提议响应(Transaction Proposal Response)，或者以其他方式转发未经过背书(Endorse)的交易(Transaction)，在确认(Validate)提交(Commit)阶段，背书策略(Endorsement Policy)将依然会被各节点(Peer)强制执行。

4. **客户端将所有背书(Endorsement)组装进交易(Transaction)**

.. image:: images/step4.png

应用程序将交易提议(Transaction Proposal)和响应(Response)组成的“交易消息(Transaction Message)”“广播”给排序服务(Ordering Service)。该交易(Transaction)将包括读写集合，所有背书节点(Endorsing Peer)的签名和频道(Channel)ID。排序服务(Ordering Service)执行其相关的操作，并不需要检查此交易的全部内容，它仅接收来自网络中所有频道(Channel)的交易(Transaction)，然后，基于频道(Channel)按时间顺序排序，并创建各个频道(Channel)的交易区块(the blocks of transactions)。

5. **交易(Transaction)被确认(Validate)并提交(Commit)**

.. image:: images/step5.png

交易区块(the blocks of transactions)被“交付”给频道(Channel)中的所有节点(Peer)。然后，对区块(Block)中的交易(Transaction)进行确认，以确保背书策略(Endorsement Policy)得到满足，并确保从账本状态(ledger state)中读取的变量集合，自从被该交易执行生成之后没有变化。最后，区块(Block)中的交易(Transaction)将被标记为有效或无效。

6. **账本(Ledger)更新**

.. image:: images/step6.png

每个节点(Peer)都会将此区块(Block)追加到所在频道(Channel)的链中，并且对于每一笔有效的交易，写集合都会被提交给当前的状态数据库(State Database)。同时，一个事件被触发，通知客户端应用程序此交易（调用）已被永久地追加到链中，以及，此交易是有效还是无效。

.. note:: 请参阅 :ref:`swimlane` 图表以更好地了解服务器端流程和 protobuffers 。

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
