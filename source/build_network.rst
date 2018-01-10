*此章节由 刘博宇 翻译，最后更新于2018.1.10* （`原文链接`_）

.. _`原文链接`: http://hyperledger-fabric.readthedocs.io/en/latest/build_network.html

构建你的第一个网络
===================

.. note:: 此文档已经被验证过，基于 "1.0.3" 版本的Docker镜像和预编译的安装实用程序。如果你使用当前主分支中的镜像和工具运行这些命令，则有可能需要修改配置或遇到错误。

“构建你的第一个网络”（BYFN）场景提供了一个示例性的 Hyperledger Fabric 网络，包括了两个组织机构(Organization)，每个组织机构(Organization)都拥有两个对等节点(Peer)，并提供“单独”的排序服务(Ordering Service)。

前提条件
----------

在我们开始之前，如果你还尚未这样做，最好检查一下是否安装了所有的 :doc:`prereqs` ，在要开发区块链应用程序或操作 Hyperledger Fabric 的平台之上。

你还需要下载并安装 :doc:`samples` 。你会注意在 ``fabric-samples`` 的源码中，包含了大量的例子。我们将使用 ``first-network`` 这个例子。现在打开这个子目录。

.. code:: bash

  cd first-network

.. note:: 本文档中所提及的命令 **必须** 在 ``fabric-samples`` 源码的 ``first-network`` 子目录中运行。如果你选择从其他位置运行命令，则提供的各种脚本将无法找到所需的可执行文件。

想要现在就运行起来？
--------------------

我们提供了一个完整注释的脚本 - ``byfn.sh`` - 利用这些Docker镜像，可以快速地启动包括2个组织机构(Organization)，4个节点(Peer)组成的 Hyperledger Fabric 网络和一个排序服务节点(orderer node)。它还将启动一个容器来运行脚本，将节点(Peer)加入到一个频道(Channel)中，部署和实例化链码(Chaincode)，并根据所部署的链码(Chaincode)来执行交易(Transaction)。

以下是 ``byfn.sh`` 脚本的帮助信息：

.. code:: bash

  ./byfn.sh -h
  Usage:
    byfn.sh -m up|down|restart|generate [-c <channel name>] [-t <timeout>]
    byfn.sh -h|--help (print this message)
      -m <mode> - one of 'up', 'down', 'restart' or 'generate'
        - 'up' - bring up the network with docker-compose up
        - 'down' - clear the network with docker-compose down
        - 'restart' - restart the network
        - 'generate' - generate required certificates and genesis block
      -c <channel name> - config name to use (defaults to "mychannel")
      -t <timeout> - CLI timeout duration in microseconds (defaults to 10000)

  Typically, one would first generate the required certificates and
  genesis block, then bring up the network. e.g.:

    byfn.sh -m generate -c <channelname>
    byfn.sh -m up -c <channelname>

如果你没有指定频道(Channel)的名称，则脚本将使用默认名称 ``mychannel`` 。CLI超时参数（使用 -t 标志）是一个可选值，如果你选择不设置它，那么你的CLI容器将在脚本结束时退出。

生成网络构件(Network Artifacts)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

准备好让它运行起来了吗？OK！执行如下的命令：

.. code:: bash

  ./byfn.sh -m generate

你将会看到一段简要的说明，描述了接下来将会发生什么，以及 yes/no 的命令行提示。按下 ``y`` 以执行相关的动作。

.. code:: bash

  Generating certs and genesis block for with channel 'mychannel' and CLI timeout of '10000'
  Continue (y/n)?y
  proceeding ...
  /Users/xxx/dev/fabric-samples/bin/cryptogen

  ##########################################################
  ##### Generate certificates using cryptogen tool #########
  ##########################################################
  org1.example.com
  2017-06-12 21:01:37.334 EDT [bccsp] GetDefault -> WARN 001 Before using BCCSP, please call InitFactories(). Falling back to bootBCCSP.
  ...

  /Users/xxx/dev/fabric-samples/bin/configtxgen
  ##########################################################
  #########  Generating Orderer Genesis block ##############
  ##########################################################
  2017-06-12 21:01:37.558 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
  2017-06-12 21:01:37.562 EDT [msp] getMspConfig -> INFO 002 intermediate certs folder not found at [/Users/xxx/dev/byfn/crypto-config/ordererOrganizations/example.com/msp/intermediatecerts]. Skipping.: [stat /Users/xxx/dev/byfn/crypto-config/ordererOrganizations/example.com/msp/intermediatecerts: no such file or directory]
  ...
  2017-06-12 21:01:37.588 EDT [common/configtx/tool] doOutputBlock -> INFO 00b Generating genesis block
  2017-06-12 21:01:37.590 EDT [common/configtx/tool] doOutputBlock -> INFO 00c Writing genesis block

  #################################################################
  ### Generating channel configuration transaction 'channel.tx' ###
  #################################################################
  2017-06-12 21:01:37.634 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
  2017-06-12 21:01:37.644 EDT [common/configtx/tool] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
  2017-06-12 21:01:37.645 EDT [common/configtx/tool] doOutputChannelCreateTx -> INFO 003 Writing new channel tx

  #################################################################
  #######    Generating anchor peer update for Org1MSP   ##########
  #################################################################
  2017-06-12 21:01:37.674 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
  2017-06-12 21:01:37.678 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
  2017-06-12 21:01:37.679 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update

  #################################################################
  #######    Generating anchor peer update for Org2MSP   ##########
  #################################################################
  2017-06-12 21:01:37.700 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
  2017-06-12 21:01:37.704 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
  2017-06-12 21:01:37.704 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update

第一步为我们所有的网络实体生成了所需的证书和密钥，以及用于启动排序服务(Ordering Service)的 ``创世区块(Genesis Block)`` 以及一组用来配置 :ref:`频道(Channel)` 的配置交易(configuration transactions)。

启动网络
^^^^^^^^^^

接下来，你可以使用如下命令来启动网络：

.. code:: bash

  ./byfn.sh -m up

再一次，你会被提示，是否要继续还是中断，回应 ``y`` ：

.. code:: bash

  Starting with channel 'mychannel' and CLI timeout of '10000'
  Continue (y/n)?y
  proceeding ...
  Creating network "net_byfn" with the default driver
  Creating peer0.org1.example.com
  Creating peer1.org1.example.com
  Creating peer0.org2.example.com
  Creating orderer.example.com
  Creating peer1.org2.example.com
  Creating cli


   ____    _____      _      ____    _____
  / ___|  |_   _|    / \    |  _ \  |_   _|
  \___ \    | |     / _ \   | |_) |   | |
   ___) |   | |    / ___ \  |  _ <    | |
  |____/    |_|   /_/   \_\ |_| \_\   |_|

  Channel name : mychannel
  Creating channel...

日志将从这里开始。这将启动所有的容器，然后完成一个完整的端到端的应用场景。成功完成后，在终端窗口中，会得到以下的报告内容：

.. code:: bash

    2017-05-16 17:08:01.366 UTC [msp] GetLocalMSP -> DEBU 004 Returning existing local MSP
    2017-05-16 17:08:01.366 UTC [msp] GetDefaultSigningIdentity -> DEBU 005 Obtaining default signing identity
    2017-05-16 17:08:01.366 UTC [msp/identity] Sign -> DEBU 006 Sign: plaintext: 0AB1070A6708031A0C08F1E3ECC80510...6D7963631A0A0A0571756572790A0161
    2017-05-16 17:08:01.367 UTC [msp/identity] Sign -> DEBU 007 Sign: digest: E61DB37F4E8B0D32C9FE10E3936BA9B8CD278FAA1F3320B08712164248285C54
    Query Result: 90
    2017-05-16 17:08:15.158 UTC [main] main -> INFO 008 Exiting.....
    ===================== Query on PEER3 on channel 'mychannel' is successful =====================

    ===================== All GOOD, BYFN execution completed =====================


     _____   _   _   ____
    | ____| | \ | | |  _ \
    |  _|   |  \| | | | | |
    | |___  | |\  | | |_| |
    |_____| |_| \_| |____/

你可以滚动浏览这些日志以查看各种交易。如果你没有看到这些，那么请查看 :ref:`疑难解答` 章节，看看我们能否帮助你发现问题的所在。

关闭网络
^^^^^^^^^^

最后，让我们把它全部都关闭掉，这样我们就可以逐步来探索网络的配置。接下来，将关闭您的容器，删除加密素材，以及四个构件(artifacts)，并从你的Docker注册表中删除链码镜像：

.. code:: bash

  ./byfn.sh -m down

再一次，你会被提示是否继续，用 ``y`` 来回应：

.. code:: bash

  Stopping with channel 'mychannel' and CLI timeout of '10000'
  Continue (y/n)?y
  proceeding ...
  WARNING: The CHANNEL_NAME variable is not set. Defaulting to a blank string.
  WARNING: The TIMEOUT variable is not set. Defaulting to a blank string.
  Removing network net_byfn
  468aaa6201ed
  ...
  Untagged: dev-peer1.org2.example.com-mycc-1.0:latest
  Deleted: sha256:ed3230614e64e1c83e510c0c282e982d2b06d148b1c498bbdcc429e2b2531e91
  ...

如果你打算了解更多底层工具和引导机制的关于信息，请继续阅读下面的章节。在接下来的部分中，我们将介绍构建完整功能 Hyperledger Fabric 网络的各个步骤和相关要求。

生成加密证书
--------------

我们使用 ``cryptogen`` 工具来为各种网络实体，生成加密素材（x509证书）。这些证书是代表了身份的标示，实体在进行通信和交易的时候，会利用这些证书来进行签名和验证身份。

它是如何工作的？
^^^^^^^^^^^^^^^^^

Cryptogen使用包含了网络拓扑结构的 ``crypto-config.yaml`` 文件，并可以为组织机构和属于该机构的组件生成一组证书和密钥。每个组织机构都配备了一个唯一的根证书（ ``ca-cert`` ），将特定的组件（节点(Peer)和排序服务节点(orderer)）绑定到该机构之上。通过为每个组织机构分配一个唯一的CA证书，我们正在模拟一个典型的网络，其中 :ref:`会员(Member)` 将使用自己的CA证书颁发机构。Hyperledger Fabric中的交易和通信，都将由实体的私钥（ ``keystore`` ）进行签名，然后通过公钥（ ``signcerts`` ）进行验证。

你会注意到文件中的 ``count`` 变量。我们用这个来指定每个组织机构中节点(Peer)的数量。在我们的案例中，每个机构有两个节点(Peer)。我们现在不会深入研究 `x.509证书和公钥基础设施 <https://en.wikipedia.org/wiki/Public_key_infrastructure>`__ 的细节。如果你有兴趣，你可以自己找时间来仔细阅读这些资料。

在运行该工具之前，让我们快速浏览一下 ``crypto-config.yaml`` 的代码片段。请特别注意 ``OrdererOrgs`` 标题下的 "Name" ， "Domain" 和 "Specs" 参数：

.. code:: bash

  OrdererOrgs:
  #---------------------------------------------------------
  # Orderer
  # --------------------------------------------------------
  - Name: Orderer
    Domain: example.com
    CA:
        Country: US
        Province: California
        Locality: San Francisco
    #   OrganizationalUnit: Hyperledger Fabric
    #   StreetAddress: address for org # default nil
    #   PostalCode: postalCode for org # default nil
    # ------------------------------------------------------
    # "Specs" - See PeerOrgs below for complete description
  # -----------------------------------------------------
    Specs:
      - Hostname: orderer
  # -------------------------------------------------------
  # "PeerOrgs" - Definition of organizations managing peer nodes
  # ------------------------------------------------------
  PeerOrgs:
  # -----------------------------------------------------
  # Org1
  # ----------------------------------------------------
  - Name: Org1
    Domain: org1.example.com

网络实体的命名约定如下 - "{{.Hostname}}.{{.Domain}}" 。因此，将我们的排序节点作为参考，我们设置了一个名为 - ``orderer.example.com`` 的排序节点，该排序节点绑定到 ``Orderer`` 的MSP ID上。该文档包含有关定义和语法的大量描述，您也可以参阅 :doc:`msp` 文档，以深入了解MSP。

运行 ``cryptogen`` 工具后，生成的证书和密钥将被保存到 ``crypto-config`` 文件夹中。

生成配置交易(Configuration Transaction)
----------------------------------------

``configtxgen`` 工具用于创建以下四个配置构件：

  * 排序节点 ``创世区块(Genesis Block)``
  * 频道 ``配置交易(Configuration Transaction)``
  * 两个 ``锚节点交易(anchor peer transactions)`` - 每个机构对应一个节点

请参阅 :doc:`configtxgen` 以获取完整的使用说明。

排序节点区块(the orderer block)是排序服务(Ordering Service)的 :ref:`创世区块(Genesis Block)` ，并且 频道交易文件(the channel transaction file) 在 :ref:`频道(Channel)` 被创建时，被广播给排序节点(the orderer)。 锚节点交易(anchor peer transactions) ，正如名字所表述的一样，指定了频道(Channel)中，每个机构的 :ref:`锚节点(Anchor Peer)` 。

它是如何工作的？
^^^^^^^^^^^^^^^^^

Configtxgen使用 ``configtx.yaml`` 文件，其包含了示例网络的定义。有三个会员 - 一个排序节点机构(Orderer Org)(``OrdererOrg``)和两个节点机构(Peer Orgs)(``Org1`` & ``Org2``)，每个管理和维护两个节点(Peer)。该文件还指定了由两个节点机构(Peer Orgs)组成的联盟 - ``SampleConsortium``。请特别注意文件顶部的 "Profiles" 部分。你会注意到有两个特别的部分，一个是排序节点创世区块(the orderer genesis
block) - ``TwoOrgsOrdererGenesis`` - 一个是频道(Channel) - ``TwoOrgsChannel``。

上面的两个很重要，因为我们会在创建构件时，将它们作为参数。

.. note:: 请注意，我们的 ``SampleConsortium`` 是在系统级配置文件中定义的，然后在频道级配置文件中被引用。频道(Channel)存在于一个联盟的范围内，所有联盟都必须在整个网络范围内进行界定。

文件还包含了其他两个值得注意的地方。首先，我们为每个节点机构(Peer Orgs)指定了锚节点(Anchor Peer)（``peer0.org1.example.com`` & ``peer0.org2.example.com``）。其次，每个会员(Member)的MSP目录位置，反过来，都可以让我们将在排序节点创世区块(the orderer genesis block)内的每个机构的根证书存储在其之中。这是一个非常重要的概念，这样任何与排序服务(Ordering Service)通信的网络实体都可以验证数字签名了。

使用工具
-----------

你可以使用 ``configtxgen`` 和 ``cryptogen`` 命令，来手动生成证书/密钥和各种配置构件。或者，您可以尝试修改 byfn.sh 脚本来实现您的目的。

手动生成构件
^^^^^^^^^^^^^

可以参考 byfn.sh 脚本中的 ``generateCerts`` 函数，以获取所需的命令，生成用于网络配置的证书，就如 ``crypto-config.yaml`` 中所定义的那样。不过，为方便起见，我们也会在这里提供参考。

首先，让我们运行 ``cryptogen`` 工具。执行文件位于 ``bin`` 目录下，所以我们需要工具所在的相对路径。

.. code:: bash

    ../bin/cryptogen generate --config=./crypto-config.yaml

你可能会看到以下的警告。但这是无害的，忽略它：

.. code:: bash

    [bccsp] GetDefault -> WARN 001 Before using BCCSP, please call InitFactories(). Falling back to bootBCCSP.

接下来，我们需要告诉 ``configtxgen`` 工具，在哪里查找需要获取的 ``configtx.yaml`` 文件。我们会告诉它在当前的目录中查找：

首先，我们需要设置一个环境变量来指定 ``configtxgen`` 应该在哪里查找 ``configtx.yaml`` 配置文件：

.. code:: bash

    export FABRIC_CFG_PATH=$PWD

然后，我们将调用 ``configtxgen`` 工具来创建排序节点创世区块(the orderer genesis block)：

.. code:: bash

    ../bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block

你可以忽略有关中间证书，证书吊销列表（crls）和MSP配置的日志警告。在这个示例网络中，我们没有使用其中的任何一个。

.. code: bash

  2017-06-12 21:01:37.562 EDT [msp] getMspConfig -> INFO 002 intermediate certs folder not found at [/Users/xxx/dev/byfn/crypto-config/ordererOrganizations/example.com/msp/intermediatecerts]. Skipping.: [stat /Users/xxx/dev/byfn/crypto-config/ordererOrganizations/example.com/msp/intermediatecerts: no such file or directory]
  2017-06-12 21:01:37.562 EDT [msp] getMspConfig -> INFO 003 crls folder not found at [/Users/xxx/dev/byfn/crypto-config/ordererOrganizations/example.com/msp/intermediatecerts]. Skipping.: [stat /Users/xxx/dev/byfn/crypto-config/ordererOrganizations/example.com/msp/crls: no such file or directory]
  2017-06-12 21:01:37.562 EDT [msp] getMspConfig -> INFO 004 MSP configuration file not found at [/Users/xxx/dev/byfn/crypto-config/ordererOrganizations/example.com/msp/config.yaml]: [stat /Users/xxx/dev/byfn/crypto-config/ordererOrganizations/example.com/msp/config.yaml: no such file or directory]

.. _createchanneltx:

创建频道配置交易(Channel Configuration Transaction)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

接下来，我们需要创建频道配置交易(Channel Configuration Transaction)构件。请务必替换 $CHANNEL_NAME 或将 CHANNEL_NAME 设置为在整个上下文环境中可使用的环境变量：

.. code:: bash

    export CHANNEL_NAME=mychannel

    # this file contains the definitions for our sample channel
    ../bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME

接下来，我们将定义频道上的 Org1 的锚节点(Anchor Peer)。同样，请确保替换 $CHANNEL_NAME 或为以下命令设置环境变量：

.. code:: bash

    ../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP

现在，我们将在同一个频道上定义 Org2 的锚节点(Anchor Peer)：

.. code:: bash

    ../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP

启动网络
-----------

We will leverage a docker-compose script to spin up our network. The
docker-compose file references the images that we have previously downloaded,
and bootstraps the orderer with our previously generated ``genesis.block``.

.. note: Before launching the network, open the ``docker-compose-cli.yaml`` file
         and comment out the script.sh in the CLI container. Your docker-compose
         should be modified to look like this:

.. code:: bash

  working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
  # command: /bin/bash -c './scripts/script.sh ${CHANNEL_NAME}; sleep $TIMEOUT'
  volumes

If left uncommented, that script will exercise all of the CLI commands when the
network is started, as we describe in the :ref:`behind-scenes` section.
However, we want to go through the commands manually in order
to expose the syntax and functionality of each call.

Pass in a moderately high value for the ``TIMEOUT`` variable (specified in seconds);
otherwise the CLI container, by default, will exit after 60 seconds.

Start your network:

.. code:: bash

          CHANNEL_NAME=$CHANNEL_NAME TIMEOUT=<pick_a_value> docker-compose -f docker-compose-cli.yaml up -d

If you want to see the realtime logs for your network, then do not supply the ``-d`` flag.
If you let the logs stream, then you will need to open a second terminal to execute the CLI calls.

.. _peerenvvars:

环境变量
^^^^^^^^^^^

For the following CLI commands against ``peer0.org1.example.com`` to work, we need
to preface our commands with the four environment variables given below.  These
variables for ``peer0.org1.example.com`` are baked into the CLI container,
therefore we can operate without passing them.  **HOWEVER**, if you want to send
calls to other peers or the orderer, then you will need to provide these
values accordingly.  Inspect the ``docker-compose-base.yaml`` for the specific
paths:

.. code:: bash

    # Environment variables for PEER0

    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    CORE_PEER_ADDRESS=peer0.org1.example.com:7051
    CORE_PEER_LOCALMSPID="Org1MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt

.. _createandjoin:

创建&添加频道(Channel)
^^^^^^^^^^^^^^^^^^^^^^^^

Recall that we created the channel configuration transaction using the
``configtxgen`` tool in the :ref:`createchanneltx` section, above. You can
repeat that process to create additional channel configuration transactions,
using the same or different profiles in the ``configtx.yaml`` that you pass
to the ``configtxgen`` tool. Then you can repeat the process defined in this
section to establish those other channels in your network.

We will enter the CLI container using the ``docker exec`` command:

.. code:: bash

        docker exec -it cli bash

If successful you should see the following:

.. code:: bash

        root@0d78bb69300d:/opt/gopath/src/github.com/hyperledger/fabric/peer#

Next, we are going to pass in the generated channel configuration transaction
artifact that we created in the :ref:`createchanneltx` section (we called
it ``channel.tx``) to the orderer as part of the create channel request.

We specify our channel name with the ``-c`` flag and our channel configuration
transaction with the ``-f`` flag. In this case it is ``channel.tx``, however
you can mount your own configuration transaction with a different name.

.. code:: bash

        export CHANNEL_NAME=mychannel

        # the channel.tx file is mounted in the channel-artifacts directory within your CLI container
        # as a result, we pass the full path for the file
        # we also pass the path for the orderer ca-cert in order to verify the TLS handshake
        # be sure to replace the $CHANNEL_NAME variable appropriately

        peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

.. note:: Notice the ``-- cafile`` that we pass as part of this command.  It is
          the local path to the orderer's root cert, allowing us to verify the
          TLS handshake.

This command returns a genesis block - ``<channel-ID.block>`` - which we will use to join the channel.
It contains the configuration information specified in ``channel.tx``.

.. note:: You will remain in the CLI container for the remainder of
          these manual commands. You must also remember to preface all commands
          with the corresponding environment variables when targeting a peer other than
          ``peer0.org1.example.com``.

Now let's join ``peer0.org1.example.com`` to the channel.

.. code:: bash

        # By default, this joins ``peer0.org1.example.com`` only
        # the <channel-ID.block> was returned by the previous command

         peer channel join -b <channel-ID.block>

You can make other peers join the channel as necessary by making appropriate
changes in the four environment variables we used in the :ref:`peerenvvars`
section, above.

安装&实例化链码(Chaincode)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note:: We will utilize a simple existing chaincode. To learn how to write
          your own chaincode, see the :doc:`chaincode4ade` tutorial.

Applications interact with the blockchain ledger through ``chaincode``.  As
such we need to install the chaincode on every peer that will execute and
endorse our transactions, and then instantiate the chaincode on the channel.

First, install the sample Go code onto one of the four peer nodes.  This command
places the source code onto our peer's filesystem.

.. code:: bash

    peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02

Next, instantiate the chaincode on the channel. This will initialize the
chaincode on the channel, set the endorsement policy for the chaincode, and
launch a chaincode container for the targeted peer.  Take note of the ``-P``
argument. This is our policy where we specify the required level of endorsement
for a transaction against this chaincode to be validated.

In the command below you’ll notice that we specify our policy as
``-P "OR ('Org0MSP.member','Org1MSP.member')"``. This means that we need
“endorsement” from a peer belonging to Org1 **OR** Org2 (i.e. only one endorsement).
If we changed the syntax to ``AND`` then we would need two endorsements.

.. code:: bash

    # be sure to replace the $CHANNEL_NAME environment variable
    # if you did not install your chaincode with a name of mycc, then modify that argument as well

    peer chaincode instantiate -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org1MSP.member','Org2MSP.member')"

See the `endorsement
policies <http://hyperledger-fabric.readthedocs.io/en/latest/endorsement-policies.html>`__
documentation for more details on policy implementation.

查询
^^^^^

Let's query for the value of ``a`` to make sure the chaincode was properly
instantiated and the state DB was populated. The syntax for query is as follows:

.. code:: bash

  # be sure to set the -C and -n flags appropriately

  peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'


调用
^^^^^^

Now let's move ``10`` from ``a`` to ``b``.  This transaction will cut a new block and
update the state DB. The syntax for invoke is as follows:

.. code:: bash

    # be sure to set the -C and -n flags appropriately

    peer chaincode invoke -o orderer.example.com:7050  --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  -C $CHANNEL_NAME -n mycc -c '{"Args":["invoke","a","b","10"]}'

查询
^^^^^

Let's confirm that our previous invocation executed properly. We initialized the
key ``a`` with a value of ``100`` and just removed ``10`` with our previous
invocation. Therefore, a query against ``a`` should reveal ``90``. The syntax
for query is as follows.

.. code:: bash

  # be sure to set the -C and -n flags appropriately

  peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

We should see the following:

.. code:: bash

   Query Result: 90

Feel free to start over and manipulate the key value pairs and subsequent
invocations.

.. _behind-scenes:

What's happening behind the scenes?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note:: These steps describe the scenario in which
          ``script.sh`` is not commented out in the
          docker-compose-cli.yaml file.  Clean your network
          with ``./byfn.sh -m down`` and ensure
          this command is active.  Then use the same
          docker-compose prompt to launch your network again

-  A script - ``script.sh`` - is baked inside the CLI container. The
   script drives the ``createChannel`` command against the supplied channel name
   and uses the channel.tx file for channel configuration.

-  The output of ``createChannel`` is a genesis block -
   ``<your_channel_name>.block`` - which gets stored on the peers' file systems and contains
   the channel configuration specified from channel.tx.

-  The ``joinChannel`` command is exercised for all four peers, which takes as
   input the previously generated genesis block.  This command instructs the
   peers to join ``<your_channel_name>`` and create a chain starting with ``<your_channel_name>.block``.

-  Now we have a channel consisting of four peers, and two
   organizations.  This is our ``TwoOrgsChannel`` profile.

-  ``peer0.org1.example.com`` and ``peer1.org1.example.com`` belong to Org1;
   ``peer0.org2.example.com`` and ``peer1.org2.example.com`` belong to Org2

-  These relationships are defined through the ``crypto-config.yaml`` and
   the MSP path is specified in our docker compose.

-  The anchor peers for Org1MSP (``peer0.org1.example.com``) and
   Org2MSP (``peer0.org2.example.com``) are then updated.  We do this by passing
   the ``Org1MSPanchors.tx`` and ``Org2MSPanchors.tx`` artifacts to the ordering
   service along with the name of our channel.

-  A chaincode - **chaincode_example02** - is installed on ``peer0.org1.example.com`` and
   ``peer0.org2.example.com``

-  The chaincode is then "instantiated" on ``peer0.org2.example.com``. Instantiation
   adds the chaincode to the channel, starts the container for the target peer,
   and initializes the key value pairs associated with the chaincode.  The initial
   values for this example are ["a","100" "b","200"]. This "instantiation" results
   in a container by the name of ``dev-peer0.org2.example.com-mycc-1.0`` starting.

-  The instantiation also passes in an argument for the endorsement
   policy. The policy is defined as
   ``-P "OR    ('Org1MSP.member','Org2MSP.member')"``, meaning that any
   transaction must be endorsed by a peer tied to Org1 or Org2.

-  A query against the value of "a" is issued to ``peer0.org1.example.com``. The
   chaincode was previously installed on ``peer0.org1.example.com``, so this will start
   a container for Org1 peer0 by the name of ``dev-peer0.org1.example.com-mycc-1.0``. The result
   of the query is also returned. No write operations have occurred, so
   a query against "a" will still return a value of "100".

-  An invoke is sent to ``peer0.org1.example.com`` to move "10" from "a" to "b"

-  The chaincode is then installed on ``peer1.org2.example.com``

-  A query is sent to ``peer1.org2.example.com`` for the value of "a". This starts a
   third chaincode container by the name of ``dev-peer1.org2.example.com-mycc-1.0``. A
   value of 90 is returned, correctly reflecting the previous
   transaction during which the value for key "a" was modified by 10.

What does this demonstrate?
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Chaincode **MUST** be installed on a peer in order for it to
successfully perform read/write operations against the ledger.
Furthermore, a chaincode container is not started for a peer until an ``init`` or
traditional transaction - read/write - is performed against that chaincode (e.g. query for
the value of "a"). The transaction causes the container to start. Also,
all peers in a channel maintain an exact copy of the ledger which
comprises the blockchain to store the immutable, sequenced record in
blocks, as well as a state database to maintain a snapshot of the current state.
This includes those peers that do not have chaincode installed on them
(like ``peer1.org1.example.com`` in the above example) . Finally, the chaincode is accessible
after it is installed (like ``peer1.org2.example.com`` in the above example) because it
has already been instantiated.

如何查看交易？
^^^^^^^^^^^^^^^

Check the logs for the CLI Docker container.

.. code:: bash

        docker logs -f cli

You should see the following output:

.. code:: bash

      2017-05-16 17:08:01.366 UTC [msp] GetLocalMSP -> DEBU 004 Returning existing local MSP
      2017-05-16 17:08:01.366 UTC [msp] GetDefaultSigningIdentity -> DEBU 005 Obtaining default signing identity
      2017-05-16 17:08:01.366 UTC [msp/identity] Sign -> DEBU 006 Sign: plaintext: 0AB1070A6708031A0C08F1E3ECC80510...6D7963631A0A0A0571756572790A0161
      2017-05-16 17:08:01.367 UTC [msp/identity] Sign -> DEBU 007 Sign: digest: E61DB37F4E8B0D32C9FE10E3936BA9B8CD278FAA1F3320B08712164248285C54
      Query Result: 90
      2017-05-16 17:08:15.158 UTC [main] main -> INFO 008 Exiting.....
      ===================== Query on PEER3 on channel 'mychannel' is successful =====================

      ===================== All GOOD, BYFN execution completed =====================


       _____   _   _   ____
      | ____| | \ | | |  _ \
      |  _|   |  \| | | | | |
      | |___  | |\  | | |_| |
      |_____| |_| \_| |____/

You can scroll through these logs to see the various transactions.

如何查看链码的日志？
^^^^^^^^^^^^^^^^^^^^

Inspect the individual chaincode containers to see the separate
transactions executed against each container. Here is the combined
output from each container:

.. code:: bash

        $ docker logs dev-peer0.org2.example.com-mycc-1.0
        04:30:45.947 [BCCSP_FACTORY] DEBU : Initialize BCCSP [SW]
        ex02 Init
        Aval = 100, Bval = 200

        $ docker logs dev-peer0.org1.example.com-mycc-1.0
        04:31:10.569 [BCCSP_FACTORY] DEBU : Initialize BCCSP [SW]
        ex02 Invoke
        Query Response:{"Name":"a","Amount":"100"}
        ex02 Invoke
        Aval = 90, Bval = 210

        $ docker logs dev-peer1.org2.example.com-mycc-1.0
        04:31:30.420 [BCCSP_FACTORY] DEBU : Initialize BCCSP [SW]
        ex02 Invoke
        Query Response:{"Name":"a","Amount":"90"}

理解 Docker Compose 的拓扑结构
--------------------------------

The BYFN sample offers us two flavors of Docker Compose files, both of which
are extended from the ``docker-compose-base.yaml`` (located in the ``base``
folder).  Our first flavor, ``docker-compose-cli.yaml``, provides us with a
CLI container, along with an orderer, four peers.  We use this file
for the entirety of the instructions on this page.

.. note:: the remainder of this section covers a docker-compose file designed for the
          SDK.  Refer to the `Node SDK <https://github.com/hyperledger/fabric-sdk-node>`__
          repo for details on running these tests.

The second flavor, ``docker-compose-e2e.yaml``, is constructed to run end-to-end tests
using the Node.js SDK.  Aside from functioning with the SDK, its primary differentiation
is that there are containers for the fabric-ca servers.  As a result, we are able
to send REST calls to the organizational CAs for user registration and enrollment.

If you want to use the ``docker-compose-e2e.yaml`` without first running the
byfn.sh script, then we will need to make four slight modifications.
We need to point to the private keys for our Organization's CA's.  You can locate
these values in your crypto-config folder.  For example, to locate the private
key for Org1 we would follow this path - ``crypto-config/peerOrganizations/org1.example.com/ca/``.
The private key is a long hash value followed by ``_sk``.  The path for Org2
would be - ``crypto-config/peerOrganizations/org2.example.com/ca/``.

In the ``docker-compose-e2e.yaml`` update the FABRIC_CA_SERVER_TLS_KEYFILE variable
for ca0 and ca1.  You also need to edit the path that is provided in the command
to start the ca server.  You are providing the same private key twice for each
CA container.

使用CouchDB
--------------

The state database can be switched from the default (goleveldb) to CouchDB.
The same chaincode functions are available with CouchDB, however, there is the
added ability to perform rich and complex queries against the state database
data content contingent upon the chaincode data being modeled as JSON.

To use CouchDB instead of the default database (goleveldb), follow the same
procedures outlined earlier for generating the artifacts, except when starting
the network pass ``docker-compose-couch.yaml`` as well:

.. code:: bash

    CHANNEL_NAME=$CHANNEL_NAME TIMEOUT=<pick_a_value> docker-compose -f docker-compose-cli.yaml -f docker-compose-couch.yaml up -d

**chaincode_example02** should now work using CouchDB underneath.

.. note::  If you choose to implement mapping of the fabric-couchdb container
           port to a host port, please make sure you are aware of the security
           implications. Mapping of the port in a development environment makes the
           CouchDB REST API available, and allows the
           visualization of the database via the CouchDB web interface (Fauxton).
           Production environments would likely refrain from implementing port mapping in
           order to restrict outside access to the CouchDB containers.

You can use **chaincode_example02** chaincode against the CouchDB state database
using the steps outlined above, however in order to exercise the CouchDB query
capabilities you will need to use a chaincode that has data modeled as JSON,
(e.g. **marbles02**). You can locate the **marbles02** chaincode in the
``fabric/examples/chaincode/go`` directory.

We will follow the same process to create and join the channel as outlined in the
:ref:`createandjoin` section above.  Once you have joined your peer(s) to the
channel, use the following steps to interact with the **marbles02** chaincode:

-  Install and instantiate the chaincode on ``peer0.org1.example.com``:

.. code:: bash

       # be sure to modify the $CHANNEL_NAME variable accordingly for the instantiate command

       peer chaincode install -n marbles -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/marbles02
       peer chaincode instantiate -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org0MSP.member','Org1MSP.member')"

-  Create some marbles and move them around:

.. code:: bash

        # be sure to modify the $CHANNEL_NAME variable accordingly

        peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble1","blue","35","tom"]}'
        peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble2","red","50","tom"]}'
        peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble3","blue","70","tom"]}'
        peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarble","marble2","jerry"]}'
        peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarblesBasedOnColor","blue","jerry"]}'
        peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["delete","marble1"]}'

-  If you chose to map the CouchDB ports in docker-compose, you can now view
   the state database through the CouchDB web interface (Fauxton) by opening
   a browser and navigating to the following URL:

   ``http://localhost:5984/_utils``

You should see a database named ``mychannel`` (or your unique channel name) and
the documents inside it.

.. note:: For the below commands, be sure to update the $CHANNEL_NAME variable appropriately.

You can run regular queries from the CLI (e.g. reading ``marble2``):

.. code:: bash

      peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["readMarble","marble2"]}'

The output should display the details of ``marble2``:

.. code:: bash

       Query Result: {"color":"red","docType":"marble","name":"marble2","owner":"jerry","size":50}

You can retrieve the history of a specific marble - e.g. ``marble1``:

.. code:: bash

      peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["getHistoryForMarble","marble1"]}'

The output should display the transactions on ``marble1``:

.. code:: bash

      Query Result: [{"TxId":"1c3d3caf124c89f91a4c0f353723ac736c58155325f02890adebaa15e16e6464", "Value":{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"tom"}},{"TxId":"755d55c281889eaeebf405586f9e25d71d36eb3d35420af833a20a2f53a3eefd", "Value":{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"jerry"}},{"TxId":"819451032d813dde6247f85e56a89262555e04f14788ee33e28b232eef36d98f", "Value":}]

You can also perform rich queries on the data content, such as querying marble fields by owner ``jerry``:

.. code:: bash

      peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarblesByOwner","jerry"]}'

The output should display the two marbles owned by ``jerry``:

.. code:: bash

       Query Result: [{"Key":"marble2", "Record":{"color":"red","docType":"marble","name":"marble2","owner":"jerry","size":50}},{"Key":"marble3", "Record":{"color":"blue","docType":"marble","name":"marble3","owner":"jerry","size":70}}]


为什么是CouchDB
-----------------

CouchDB is a kind of NoSQL solution. It is a document oriented database where document fields are stored as key-value mpas. Fields can be either a simple key/value pair, list, or map.
In addition to keyed/composite-key/key-range queries which are supported by LevelDB, CouchDB also supports full data rich queries capability, such as non-key queries against the whole blockchain data,
since its data content is stored in JSON format and fully queryable. Therefore, CouchDB can meet chaincode, auditing, reporting requirements for many use cases that not supported by LevelDB.

CouchDB can also enhance the security for compliance and data protection in the blockchain. As it is able to implement field-level security through the filtering and masking of individual attributes within a transaction, and only authorizing the read-only permission if needed.

In addition, CouchDB falls into the AP-type (Availability and Partition Tolerance) of the CAP theorem. It uses a master-master replication model with ``Eventual Consistency``.
More information can be found on the
`Eventual Consistency page of the CouchDB documentation <http://docs.couchdb.org/en/latest/intro/consistency.html>`__.
However, under each fabric peer, there is no database replicas, writes to database are guaranteed consistent and durable (not ``Eventual Consistency``).

CouchDB is the first external pluggable state database for Fabric, and there could and should be other external database options. For example, IBM enables the relational database for its blockchain.
And the CP-type (Consistency and Partition Tolerance) databases may also in need, so as to enable data consistency without application level guarantee.

关于数据持久化的备注说明
------------------------

If data persistence is desired on the peer container or the CouchDB container,
one option is to mount a directory in the docker-host into a relevant directory
in the container. For example, you may add the following two lines in
the peer container specification in the ``docker-compose-base.yaml`` file:

.. code:: bash

       volumes:
        - /var/hyperledger/peer0:/var/hyperledger/production

For the CouchDB container, you may add the following two lines in the CouchDB
container specification:

.. code:: bash

       volumes:
        - /var/hyperledger/couchdb0:/opt/couchdb/data

.. _疑难解答:

疑难解答
----------

-  Always start your network fresh.  Use the following command
   to remove artifacts, crypto, containers and chaincode images:

   .. code:: bash

      ./byfn.sh -m down

   .. note:: You **will** see errors if you do not remove old containers
             and images.

-  If you see Docker errors, first check your docker version (:doc:`prereqs`),
   and then try restarting your Docker process.  Problems with Docker are
   oftentimes not immediately recognizable.  For example, you may see errors
   resulting from an inability to access crypto material mounted within a
   container.

   If they persist remove your images and start from scratch:

   .. code:: bash

       docker rm -f $(docker ps -aq)
       docker rmi -f $(docker images -q)

-  If you see errors on your create, instantiate, invoke or query commands, make
   sure you have properly updated the channel name and chaincode name.  There
   are placeholder values in the supplied sample commands.


-  If you see the below error:

   .. code:: bash

       Error: Error endorsing chaincode: rpc error: code = 2 desc = Error installing chaincode code mycc:1.0(chaincode /var/hyperledger/production/chaincodes/mycc.1.0 exits)

   You likely have chaincode images (e.g. ``dev-peer1.org2.example.com-mycc-1.0`` or
   ``dev-peer0.org1.example.com-mycc-1.0``) from prior runs. Remove them and try
   again.

   .. code:: bash

       docker rmi -f $(docker images | grep peer[0-9]-peer[0-9] | awk '{print $3}')

-  If you see something similar to the following:

   .. code:: bash

      Error connecting: rpc error: code = 14 desc = grpc: RPC failed fast due to transport failure
      Error: rpc error: code = 14 desc = grpc: RPC failed fast due to transport failure

   Make sure you are running your network against the "1.0.0" images that have
   been retagged as "latest".

-  If you see the below error:

   .. code:: bash

     [configtx/tool/localconfig] Load -> CRIT 002 Error reading configuration: Unsupported Config Type ""
     panic: Error reading configuration: Unsupported Config Type ""

   Then you did not set the ``FABRIC_CFG_PATH`` environment variable properly.  The
   configtxgen tool needs this variable in order to locate the configtx.yaml.  Go
   back and execute an ``export FABRIC_CFG_PATH=$PWD``, then recreate your
   channel artifacts.

-  To cleanup the network, use the ``down`` option:

   .. code:: bash

       ./byfn.sh -m down

-  If you see an error stating that you still have "active endpoints", then prune
   your Docker networks.  This will wipe your previous networks and start you with a
   fresh environment:

   .. code:: bash

        docker network prune

   You will see the following message:

   .. code:: bash

      WARNING! This will remove all networks not used by at least one container.
      Are you sure you want to continue? [y/N]

   Select ``y``.

.. note:: If you continue to see errors, share your logs on the
          **fabric-questions** channel on
          `Hyperledger Rocket Chat <https://chat.hyperledger.org/home>`__
          or on `StackOverflow <https://stackoverflow.com/questions/tagged/hyperledger-fabric>`__.

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
