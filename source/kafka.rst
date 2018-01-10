*此章节由 刘博宇 翻译，最后更新于2018.1.9* （`原文链接`_）

.. _`原文链接`: http://hyperledger-fabric.readthedocs.io/en/latest/kafka.html

基于 Kafka 的排序服务(Ordering Service)
========================================

前言
------

本文假设读者已经知道如何配置Kafka集群和ZooKeeper集合。本指南的目的是帮助您了解所需执行的步骤，以使一组 Hyperledger Fabric 排序服务节点（Ordering Service Node - OSN）可以使用您的Kafka群集，并向区块链网络提供排序服务。

重点
------

每一个频道(Channel)都会映射到Kafka中的一个独立的单分区主题(a separate single-partition topic)。当排序服务节点（Ordering Service Node - OSN）通过 ``广播(Broadcast)`` RPC接收到交易(Transaction)时，它会进行检查以确保广播客户端拥有写入频道(Channel)的权限，然后，将这些交易(Transaction)转发(relay)（即，生产）至Kafka中的适当分区中。这个分区也会被排序服务节点（Ordering Service Node - OSN）使用，OSN将接收到的交易(Transaction)分组打包成本地区块(Block)，然后，将它们保存在本地的帐本(Ledger)中，并通过 ``分发(Deliver)`` RPC 发送给接收的客户端。对于低层细节，请参考文档 `基于Kafka的Fabric排序服务 <http://wutongtree.github.io/translations/Kafka-based-Ordering-Service_zh>`_ ( `英文原文 <https://docs.google.com/document/d/1vNMaM7XhOlu9tB_10dKnlrhy5d7b1u8lSY8a-kVjCO4/edit>`_ ) - 图8是对上述过程的概要表述。

步骤
-----

设 ``K`` 和 ``Z`` 分别为Kafka集群和ZooKeeper集合的节点数量：

#. ``K`` 应该至少设置为4。（正如我们将在下面的步骤4中所解释的，这是宕机故障容错所必需的最小节点数量，即，对于4个代理人(Broker)，可以有1个代理人(Broker)宕机，所有频道(Channel)将仍然可以继续读写，并且也可以创建新的频道(Channel)。）

#. ``Z`` 可以是3,5或7。它必须是奇数，以避免令人头痛(split-brain)的情况发生，并且，为了避免单点故障，必须大于1。如果超过7台ZooKeeper服务器，则是完全没有必要的。

然后按以下步骤进行：

3. 排序服务(Orderers): **对网络中创世区块(Genesis Block)里的Kafka相关信息进行编码** ，如果您使用 ``configtxgen`` ,请编辑 ``configtx.yaml`` ，或为系统频道的创世区块，选择一个预设的配置文件。 

   #. ``Orderer.OrdererType`` 设置为 ``kafka`` 。
   #. ``Orderer.Kafka.Brokers`` 需要包括 *至少两个* 在集群中的Kafka代理的节点地址，以 ``IP:port`` 的形式。该清单并不需要列出所有的节点地址。（这只是引导启动节点。）

#. 排序服务(Orderers): **设置最大的区块大小** ，每个区块最大为 `Orderer.AbsoluteMaxBytes` 个字节（不包括区块头），您可以在 ``configtx.yaml`` 中设置这个值。将这个值标记为 ``A`` ，并记下它。这将影响您如何在步骤6中配置Kafka代理。

#. 排序服务(Orderers): **创建创世区块** ，使用 ``configtxgen`` 。您在上面的步骤3和步骤4中的配置是系统范围的配置，即它们适用于整个OSN网络。然后，记下创世区块的位置。

#. Kafka集群: **正确地配置您的Kafka代理** ，确保每一个Kafka代理，都进行了如下的关键配置：

   #. ``unclean.leader.election.enable = false`` — 数据一致性是区块链环境的关键。我们不能在同步副本集合之外选择频道的领导者，否则我们面临前任领导者生成的内容被重写的风险，并且因此导致重写排序节点生成的区块链。

   #. ``min.insync.replicas = M`` — 选择一个值 ``M`` ，使得 ``1 < M < N`` （请参阅下面的 ``default.replication.factor`` ）。当数据被写入至少 ``M`` 个副本时，数据被视为已提交，（然后将其视为同步并从属于同步副本集合或ISR）。在任何其他情况下，写操作都会返回一个错误。然后：

      #. 如果 ``N-M`` 个副本不可用时，操作可以正常进行。
      #. 如果更多的副本变得不可用，Kafka将不能维持 ``M`` 的ISR集合，所以它将停止写入。读取没有问题。当 ``M`` 个副本进入同步状态时，频道将再次变为可写入状态。

   #. ``default.replication.factor = N`` — 选择一个值 ``N`` ，使得 ``N < K`` 。副本因子 ``N`` 表示每个频道将其数据复制给 ``N`` 个代理。这些是频道ISR集合的候选者。正如我们在上面的 ``min.insync.replicas section`` 部分中提到的，并非所有代理都必须始终可用。 ``N`` 应该设置为 *严格小于* ``K``，因为如果少于 ``N`` 个代理，就无法创建频道。因此，如果设置 ``N = K`` ，则单个代理停机，意味着在区块链网络中将不能创建新的频道 - 排序服务的崩溃容错功能将不复存在。

      根据上面所描述的， ``M`` 和 ``N`` 的最小值分别是2和3。 这个配置能保证，如果存在异常，将能依然允许创建新的频道，并保证所有的频道仍然是可写的。

   #. ``message.max.bytes`` 和 ``replica.fetch.max.bytes`` 应设置为大于 ``A`` 的值, 即上面步骤4中，您在 ``Orderer.AbsoluteMaxBytes`` 中设置的值。为账户头添加一些缓冲的空间，1 MiB是足够用的。以下适用：

      ::

         Orderer.AbsoluteMaxBytes < replica.fetch.max.bytes <= message.max.bytes

      （为了完整性，我们注意到 ``message.max.bytes`` 应该严格的小于 ``socket.request.max.bytes`` ，默认设置为100 MiB。如果你希望区块大于100 MiB，你需要修改 ``fabric/orderer/kafka/config.go`` 中 ``brokerConfig.Producer.MaxMessageBytes`` 的值，并从源码重新进行编译。不建议这么做。）

   #. ``log.retention.ms = -1`` - 在排序服务(Ordering Service)添加对裁剪Kafka日志的支持之前，您应该禁用基于时间的留存并防止片段(segments)过期。（基于文件大小的留存 - 请参阅 ``log.retention.bytes`` - 写这篇文章时，在Kafka中是默认禁用的，所以不需要明确设置。）

#. 排序服务(Orderers): **将每个OSN都指向创世区块** ，编辑 ``orderer.yaml`` 中的 ``General.GenesisFile`` ，以使其指向步骤5中所创建的创始区块。（在此过程中，确保YAML文件中，其他的所有键值，都被正确的设置过。）

#. 排序服务(Orderers): **调整轮询间隔和超时** （可选步骤）

   #. ``orderer.yaml`` 文件中的 ``Kafka.Retry`` 一节，允许您调整 元数据/生产者/消费者 请求的频率，以及 socket 超时。（这也是Kafka生产者或消费者的所有配置。）
   #. 另外，当一个新的频道(Channel)被创建，或者当一个现有的(Channel)被重新加载时（例如，刚刚重新启动的排序服务(orderer)），排序服务(orderer)将以下面的方式与Kafka集群进行交互：

      #. 它为该频道(Channel)所对应的Kafka分区，创建一个Kafka生产者（写入者）。
      #. 它使生产者发布一个 ``CONNECT`` 消息到该分区。
      #. 它为该分区创建一个Kafka消费者（读取者）。

      如果这些步骤，有任何一个失败，您可以调整重复的频率。具体来说，他们将会不断地重新尝试在 ``Kafka.Retry.ShortTotal`` 中，以 ``Kafka.Retry.ShortInterval`` 为间隔，然后，在 ``Kafka.Retry.LongTotal`` 中，以 ``Kafka.Retry.LongInterval`` 为间隔，直到成功为止。请注意，直到上述所有步骤都成功完成之前，排序服务(orderer)将无法写入或读取频道(Channel)。

#. **设置OSNs和Kafka集群，以使其可以通过SSL进行通信** （可选步骤，但强烈建议），请参阅 `the Confluent guide <http://docs.confluent.io/2.0.0/kafka/ssl.html>`_ 以更进一步了解Kafka集群，在每个OSN的 ``orderer.yaml`` 中，为 ``Kafka.TLS`` 进行设置。

#. **按照如下顺序启动各服务节点：ZooKeeper集合，Kafka集群，排序服务节点(Ordering Service Node)**

其他注意事项
-------------------------

#. **首选区块大小(Preferred Message Size)** 。在上面的步骤4中（参见 `步骤`_ 一节），您也可以通过设置 ``Orderer.Batchsize.PreferredMaxBytes`` ，来配置区块(Block)的大小。Kafka在处理相对较小的消息时，可以提供更高的吞吐量，其设计目标为不超过1MiB。

#. **使用环境变量来覆盖配置** 。当使用Fabric提供的示例 Kafka 和 Zookeeper 的Docker镜像时（分别参见 ``images/kafka`` 和 ``images/zookeeper`` ），您可以使用环境变量，来覆盖Kafka代理或ZooKeeper服务器的配置。用 ``_`` 替换属性名称里的 ``.`` ，例如 ``KAFKA_UNCLEAN_LEADER_ELECTION_ENABLE=false`` 将允许您覆盖 ``unclean.leader.election.enable`` 的默认值。OSN的 *本地配置* 也同样如此，也就是说，可以在 ``orderer.yaml`` 中进行配置。例如，``ORDERER_KAFKA_RETRY_SHORTINTERVAL=1s`` 允许您覆盖 ``Orderer.Kafka.Retry.ShortInterval`` 的默认值。

支持的Kafka版本
-----------------

Fabric使用 `sarama客户端库 <https://github.com/Shopify/sarama>`_ 并支持以下的Kafka客户端版本：

* ``版本: 0.9.0``
* ``版本: 0.10.0``
* ``版本: 0.10.1``
* ``版本: 0.10.2``

Fabric提供的示例Kafka服务器映像包含了Kafka服务端版本 ``0.10.2`` 。Fabric的排序服务节点(Ordering Service Node - OSN)内置了Kafka客户端，并进行了默认配置，以匹配此版本，可以直接投入使用。如果您没有使用Fabric提供的示例Kafka服务器映像，请确保 ``orderer.yaml`` 中的 ``Kafka.Version`` 属性，进行了相关的配置，以兼容您的Kafka服务端版本。

调试
------

设置 ``General.LogLevel`` 为 ``DEBUG`` ，并在 ``orderer.yaml`` 中，设置 ``Kafka.Verbose`` 为 ``true`` .

例子
------

Sample Docker Compose的配置文件中，包含了上面推荐的设置，可以在目录 ``fabric/bddtests`` 中找到。查找 ``dc-orderer-kafka-base.yml`` 和 ``dc-orderer-kafka.yml`` 。

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
