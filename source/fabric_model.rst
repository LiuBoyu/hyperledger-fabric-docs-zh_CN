*此章节由 刘博宇 翻译，最后更新于2018.1.3* （`原文链接`_）

.. _`原文链接`: http://hyperledger-fabric.readthedocs.io/en/latest/fabric_model.html

Hyperledger Fabric 模型
========================

本节概述了 Hyperledger Fabric 的关键设计特征，并介绍了其是如何履行其全面而可定制的企业区块链解决方案的承诺的。

* :ref:`资产(Assets)` - 资产(Assets)的定义使得在网络上可以交换几乎所有具有货币价值的东西，从食物到古董车到货币期货。
* :ref:`链码(Chaincode)` - 链码(Chaincode)包括，交易排序，限制所需的信任级别，跨节点类型的验证，优化网络可扩展性和性能。
* :ref:`帐本功能(Ledger Features)` - 不可变的共享账本，记录了一个频道(Channel)的所有交易历史记录，并包含类似于SQL的查询功能，以有效审计和解决争议。
* :ref:`基于频道(Channel)的隐私保护` - 频道(Channel)使多边交易具有高度的隐私性和机密性，并使竞争的企业和受管制的行业在共同的网络上交换资产。
* :ref:`安全及会员服务(Security-Membership-Services)` - 具有权限的会员提供了一个受信任的区块链网络，在这个网络中，参与者知道所有交易都可以被授权的监管机构和审计人员发现并追查。
* :ref:`共识(Consensus)` - 统一的共识(Consensus)使企业具备所需的灵活性和可伸缩性。

.. _资产(Assets):

资产(Assets)
-------------

资产(Assets)可以从有形资产（不动产和硬件）到无形资产（合同和知识产权）。Hyperledger Fabric提供了使用链码交易(Chaincode Transaction)修改资产(Assets)的能力。

在 Hyperledger Fabric 中，资产(Assets)被表示为一组键值对集合，其状态更改的历史记录做为“交易”被记录在 :ref:`频道(Channel)` 的账本之上。资产(Assets)可以用二进制和JSON形式表示。

你可以在 Hyperledger Fabric 应用程序中轻松定义和使用资产，使用 `Hyperledger Composer <https://github.com/hyperledger/composer>`_ 。

.. _链码( Chaincode ):

链码(Chaincode)
----------------

链码(Chaincode)是一段代码，它定义一个或多个资产(Assets)，以及修改资产的相关交易指令。换言之，就是业务逻辑。链码(Chaincode)按执行规则读取或修改键值对，或状态数据库(State Database)信息。链码(Chaincode)函数基于账本的当前状态数据库(State Database)执行，当有交易提议(Transaction Proposal)时启动。链码(Chaincode)执行后产生一组键值的写入操作（写集），可以提交给网络并应用于所有节点(Peer)的帐本(Ledger)。

.. _帐本功能(Ledger Features):

帐本功能(Ledger Features)
-------------------------

账本(Ledger)是一组顺序的、防篡改的记录集，记录了 Fabric 中所有的状态改变(State Transition)。状态改变(State Transition)是链码(Chaincode)调用（“交易”）的结果，由相关的参与者提交。每笔交易都会产生一组资产(Assets)的键值对集合，并将其提交给账本(Ledger)，进行创建，更新或删除。

账本(Ledger)由一个区块链（“链”）和一个状态数据库组成，区块链用于存储不可变的顺序记录块，状态数据库用来维护当前的数据状态。每一个账本(Ledger)对应一个频道(Channel)。每个节点(Peer)为其所属的每个频道(Channel)保留一份账本(Ledger)的副本。

- 基于键(Key)查询和更新账本(Ledger)，范围查询和组合键查询
- 基于丰富查询语言(a rich query language)的只读查询（如果使用 CouchDB 作为状态数据库）
- 只读历史记录查询 - 基于键(Key)的历史记录查询，可以支持数据溯源场景
- 交易(Transaction)由在链码(Chaincode)中读取的各个版本的键值集合（读集）和写入的各个版本的键值集合（写集）组成
- 交易(Transaction)包含每个背书节点(Endorsing Peer)的签名，并提交给排序服务(共识服务)(Ordering Service)
- 交易(Transaction)被顺序打包进区块(Block)中，并从排序服务(共识服务)(Ordering Service)“交付”给同一频道(Channel)的其他节点(Peer)
- 节点(Peer)基于背书策略(Endorsement Policy)来验证交易并执行
- 在追加进区块(Block)之前，会进行版本检查(Versioning Check)，以确保，在链码(Chaincode)执行期间，读取的资产(Assets)状态未发生变化
- 交易一旦得到确认和提交，将不可改变
- 每个频道(Channel)的账本(Ledger)都包含了一个配置区块(Configuration Block)，用于定义策略，访问控制列表和其他相关信息
- 频道(Channel)包含了 :ref:`MSP` 实例，允许从不同的证书颁发机构派生加密资料

请参阅 :doc:`ledger` 主题，以深入了解数据库，存储结构和“查询能力”。

.. _基于频道(Channel)的隐私保护:

基于频道(Channel)的隐私保护
---------------------------

Hyperledger Fabric在每个频道(Channel)的基础上使用了一个不可变的帐本，以及可以操纵和修改资产当前状态的链码(Chaincode)（例如：更新键值对）。帐本存在于频道(Channel)的范围之内 - 它可以共享给整个网络（假设所有参与者都在一个共同的频道上运行），又或者，也可以私有化，只包含一组特定的参与者。

在后一种情况下，这些参与者将创建一个单独的频道(Channel)，从而隔离他们的交易和帐本。为了解决透明度与隐私之间的矛盾，链码(Chaincode)只需安装在需要访问资产(Assets)状态的节点上，执行读取和写入操作（换言之，如果链码(Chaincode)没有安装在某节点之上，则此节点将不能访问此账本）。或更进一步，混淆数据，链码(Chaincode)中的数据（部分或全部）在追加到账本之前，可以使用常见的加密算法（例如AES）进行加密。

.. _安全及会员服务(Security-Membership-Services):

安全及会员服务(Security-Membership-Services)
--------------------------------------------

Hyperledger Fabric巩固了所有参与者都拥有已知身份的交易网络。公钥基础设施(Public Key Infrastructure - PKI)用于生成加密证书，加密证书与组织机构，网络组件、最终用户或客户端应用相绑定。因此，数据访问控制可以在更广泛的网络和渠道层面进行管理和维护。在 Hyperledger Fabric 中，这个“具有权限的(permissioned)”的概念与“频道(channel)”的存在和能力相关联，这有助于解决将隐私性和机密性放在首要位置的场景。

请参阅 :doc:`msp` 主题以更好地理解 Hyperledger Fabric 的加密实现及相关的签名，校验，鉴权方法。

.. _共识( Consensus ):

共识(Consensus)
----------------

在分布式账本技术中，共识(Consensus)最近已成为一个单一函数内特定算法的词汇。然而，共识(Consensus)所包含的含义更多，不仅仅是简单地共同商议交易顺序。这在 Hyperledger Fabric 的整个交易流程中，非常突出地表现出来，从提议(Proposal)和背书(Endorsement)，到排序(Ordering)，确认(Validation)和提交(Commitment)。简而言之，共识(Consensus)被定义为一个区块内交易集合的正确性闭环校验。

当一个区块内所有交易的顺序和结果都已经明确地按策略标准检查后，共识(Consensus)最终达成。这些检查发生在交易的整个生命周期，包括使用背书策略(Endorsement Policy)来决定哪些特定的会员(Member)必须背书(Endorse)某个特定的交易类型，以及，使用系统链码(System Chaincode)来确保这些策略得到执行和维护。提交之前，节点们(Peers)将调用这些系统链码(System Chaincode)，以确保获得来自适当实体(The Appropriate Entities)的足够数量的背书(Endorsement)。而且，在任何一个包含交易的区块被追加进账本之前，都会进行版本检查(Versioning Check)，以使账本的当前状态达成一致。最后的这步检查提供非常必要的保护，以避免双重支出操作和其他可能危及数据完整性的威胁，并允许针对非静态变量的函数执行。

除了大量的背书(Endorsement)，确认(Validity)和版本检查(Versioning Check)之外，身份验证也同时在交易流程的所有方向进行着。访问控制列表在分层的网络结构上实现（排序服务(Ordering Service)到频道(Channel)），并且，当交易提议(Transaction Proposal)通过不同的构件时，负载数据将被多次签名(Signed)，验证(Verified)和认证(Authenticated)。总而言之，共识(Consensus)不仅限于代表一组批量交易的商议顺序，还包括了发生在整个交易的过程期间的，从提议(Proposal)到提交(Commitment)的持续不断的各种校验。

查看 :doc:`txflow` 图表以更直观的理解共识。

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
