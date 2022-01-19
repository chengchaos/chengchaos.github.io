---
title: How Fabric networks are structured
key: 2022-01-05
tags: Fabric fabric-networks
---



参考： [https://hyperledger-fabric.readthedocs.io/en/latest/network/network.html](https://hyperledger-fabric.readthedocs.io/en/latest/network/network.html)

<!--more-->

This topic will describe, **at a conceptual level**, how  Hyperledger Fabric allows organizations to collaborate in the formation  of blockchain networks. If you’re an architect, administrator or  developer, you can use this topic to get a solid understanding of the  major structure and process components in a Hyperledger Fabric  blockchain network. This topic will use a manageable example that  introduces all of the major components in a blockchain network.

After reading this topic and understanding the concept of policies, you  will have a solid understanding of the decisions that organizations need to make to establish the policies that control a deployed Hyperledger  Fabric network. You’ll also understand how organizations manage network  evolution using declarative policies – a key feature of Hyperledger  Fabric. In a nutshell, you’ll understand the major technical components  of Hyperledger Fabric and the decisions organizations need to make about them.

Note: in this topic, we’ll refer to the structure of a network that does not have a “system channel”, a channel run by the ordering service that ordering nodes are bootstrapped with. For a version of this topic that  does use the system channel, check out [Blockchain network](https://hyperledger-fabric.readthedocs.io/en/release-2.2/network/network.html).

## What is a blockchain network?

A blockchain network is a technical infrastructure that provides ledger  and smart contract (which are packaged as part of a “chaincode”)  services to applications. Primarily, smart contracts are used to  generate transactions which are subsequently distributed to every peer  node in the network where they are immutably recorded on their copy of  the ledger. The users of applications might be end users using client  applications or blockchain network administrators.

在大多数情况下，多个组织聚在一起形成一个**通道**，在该通道上调用 chaincodes 上的交易，并且权限由一组策略决定，这些策略在最初配置通道时达成一致。此外，根据组织的协议，政策可以随着时间的推移而改变。

**在这个主题中，我们将同时提到“网络”和“渠道”。在Hyperledger Fabric中，这些术语实际上是同义的，因为它们共同指的是组织、组件、策略和流程，它们在一个已定义的结构中管理组织之间的交互。**

## The sample network

Before we start, let’s show you what we’re aiming at! Here’s a diagram representing the **final state** of our sample network.

现在看起来可能很复杂，但是当我们讨论这个话题的时候，我们将逐步构建网络，这样您就可以看到组织 R1、R2 和 R0 是如何为网络提供基础设施来帮助形成网络的。此基础设施实现了区块链网络，并且由组成网络的组织(例如，可以添加新组织)同意的策略进行治理。您将了解应用程序如何使用区块链网络提供的账本和智能合约服务。

![x](https://hyperledger-fabric.readthedocs.io/en/latest/_images/network.diagram.1.png)



R1、R2、R0三个组织共同决定建立一个网络。这个网络有一个配置 CC1，所有的组织都同意这个配置，它列出了组织的定义以及定义每个组织将在 channel 上扮演的角色的策略。

在这个 channel 上，R1 和 R2 将把名为 P1 和 P2 的 peers 加入到 channel C1，而R0拥有 channel 的 ordering server  O。所有这些节点都将包含 channel 的 ledger (L1)的副本，在这里记录 transactions。请注意，ordering service 保存的分类账副本不包含状态数据库。R1 和 R2 还将通过它们拥有的应用程序 A1 和 A2 与 channel 交互。这三个组织都有一个证书颁发机构，该机构为其组织的节点、管理员、组织定义和应用程序生成了必要的证书。

## Creating the network

The first step in creating a network or a channel is to agree to and then define its configuration:

![creaating the network](https://hyperledger-fabric.readthedocs.io/en/latest/_images/network.diagram.2.png)

channel 配置 CC1 已经得到组织 R1、R2 和 R0 的同意，并且包含在一个称为 "configuration block" 的块中，该块通常是由 `configtxgen` 工具从 `configtx.yaml` 创建的。虽然一个组织可以单方面创建一个 channel ，然后邀请其他组织加入(我们将在 [Adding an organization to an existing channel](https://hyperledger-fabric.readthedocs.io/en/latest/network/network.html#adding-an-organization-to-an-existing-channel) 进行探讨)，但现在我们假设这些组织从一开始就希望在该渠道上进行合作。

一旦配置块存在，通道就可以说是**逻辑上存在的**，即使没有任何组件在物理上连接到它。这个配置块包含可以连接组件并在通道上交互的组织的记录，以及定义如何做出决策和达成特定结果的结构的**策略(policies)**。虽然 peers 和应用程序是网络中的关键角色，但它们在通道中的行为更多地由通道配置策略决定，而不是其他任何因素。有关策略以及它们如何在通道配置中定义的更多信息，请参阅 [Policies](https://hyperledger-fabric.readthedocs.io/en/latest/policies/policies.html).

这些组织的定义及其管理员的身份必须由与每个组织关联的证书颁发机构(CA)创建。在我们的示例中，组织 R1、R2 和 R0 的认证和组织定义分别由 CA1、CA2 和 CA0 创建。有关如何创建CA的信息，请参阅 [Planning for a CA](https://hyperledger-fabric-ca.readthedocs.io/en/latest/deployguide/ca-deploy-topology.html) 。创建 CA 后，请参阅 [Registering and enrolling identities with a CA](https://hyperledger-fabric-ca.readthedocs.io/en/latest/deployguide/use_CA.html) ，以了解如何定义组织和为管理员和节点创建身份。

For more information about using `configtxgen` to create a configuration block, check out [Using configtx.yaml to build a channel configuration](https://hyperledger-fabric.readthedocs.io/en/latest/create_channel/create_channel_config.html).

### Certificate Authorities /  证书颁发机构

证书颁发机构在网络中扮演着关键角色 *[ play a key role in ]*，因为它们分发 X.509 证书，可用于标识属于某个组织的组件。由认证机构签发的证书也可以用来签署交易，表明该组织认可 *[ endorses ]* 交易结果——这是交易结果被接受进入分类账的先决条件。让我们更详细地 *[ in a little more detail ]* 研究一下 CA 的这两个方面 *[ aspects ]*。

首先，区块链网络的不同组件使用证书相互标识自己来自特定的组织。这就是为什么通常有一个以上的CA支持区块链网络——不同的组织经常使用不同的CA。我们将在信道中使用三个ca;每个组织一个。事实上，CA是如此重要，Hyperledger Fabric 为您提供了一个内置的 CA (称为 Fabric-CA )来帮助您开始工作，尽管在实践中，组织将选择使用他们自己的 CA。

证书到成员组织的映射是通过一个叫做会员服务提供者( Membership Services Provider MSP )的结构实现的，它是由根 CA 通过创建一个 MSP 绑定到一个根 CA 证书识别组件和身份定义了一个组织。通道配置 ( channel  configuration) 可以通过策略 (a policy) 将某些权利和权限分配给该组织(这将给一个特定的组织,如R1、向频道添加新组织的权利)。我们没有在这些图上显示 MSPs，因为它们会把它们弄乱，但是因为它们定义了组织，所以它们非常重要。

其次，我们将在后面看到由 CA 颁发的证书如何处于事务生成和验证过程的核心。具体来说 Specifically，X.509 证书被用于客户端应用程序的交易提议 transaction proposals 和智能合约交易响应 transaction responses 中以进行数字签名交易。随后，托管分类帐副本的网络节点在接受分类帐上的交易之前验证交易签名是有效的。

## Join nodes to the channel

Peers 是网络的基本元素，因为它们托管账簿 host ledgers 和链码 chaincode (包含智能合约 smart contracts)，因此是在一个通道上进行交易的组织连接到该通道的物理点之一(另一个是应用程序)。一个 Peer 可以属于组织认为合适的任意多个通道(取决于某些因素，如 peer pod 的处理限制和特定国家存在的数据驻留规则)。有关对等点的更多信息，请查看 [Peers](https://hyperledger-fabric.readthedocs.io/en/latest/peers/peers.html)。

另一方面 *[ on the other hand ]* ，排序服务 *[ ordering service ]* 从应用程序中收集认可的交易 *[ endorsed transactions ]*，并将其放到交易块中 *[ orders them into ]*，这些交易块随后被分发到通道中的每个 peer 节点。在每一个提交的 peers 上，交易被记录下来，分类帐的本地副本被适当地更新。排序服务对于特定的通道是唯一的，服务该通道的节点也称为“一致性集 *[ consenter set ]*”。即使一个节点(或一组节点)为多个通道提供服务，每个通道的 Order service 也被认为是 Ordering services 的一个不同实例。有关订购服务的更多信息，请参阅 [The Ordering Service](https://hyperledger-fabric.readthedocs.io/en/latest/orderer/ordering_service.html)。

**For information about how to create peer and ordering nodes, check out [Deploying a production network](https://hyperledger-fabric.readthedocs.io/en/latest/deployment_guide_overview.html).**

Because R1, R2, and R0 are listed in the channel configuration, they are allowed to join peers (in the case of R1 and R2) or ordering nodes (in  the case of R0) to the channel.

![..](https://hyperledger-fabric.readthedocs.io/en/latest/_images/network.diagram.3.png)



R1 的对等点 P1 和 R2 的对等点 P2，以及 R0 的 Ording Service O，通过 [Create a channel](https://hyperledger-fabric.readthedocs.io/en/latest/create_channel/create_channel_participation.html) 教程中描述的过程加入通道。请注意，虽然只有一个 ording 节点(1)连接到该通道，但在生产场景中，an ordering service 应该至少包含三个节点。然而，就本主题而言，概念化 an ordering service 和网络其他组件的交互比理解高可用性需求如何影响配置决策更重要。属于每个组织的节点拥有由与该组织相关联的证书颁发机构为其创建的 x.509 证书。P1 的证书由 CA1 创建，P2 的证书由 CA2 创建，依此类推 and so on。

通道中的每个节点存储通道的 ledger L1 的副本，该副本将随每个新块更新(请注意，ordering service 只包含 ledger 的区块链部分，而不包含  [state database](https://hyperledger-fabric.readthedocs.io/en/latest/glossary.html#state-database))。因此，我们可以把 L1 看作物理上托管在 P1 上，但逻辑上托管在通道 C1 上。最好的做法是 R1和 R2 将它们的对等体 P1 和 P2 作为 [anchor peers](https://hyperledger-fabric.readthedocs.io/en/latest/glossary.html#anchor-peer)，因为这将引导 R1 和 R2 之间的网络通信。

After the ordering service has been joined to the channel, it is  possible to propose and commit updates to the channel configuration, but little else *[ 但也仅此而已/但除此之外就没什么了 ]* . Next, you must install, approve, and commit a chaincode on a channel.

## Install, approve, and commit a chaincode

Chaincodes are installed on peers, and then defined and committed on a channel:

![xx](https://hyperledger-fabric.readthedocs.io/en/latest/_images/network.diagram.4.png)



在 Fabric 中，定义 peer 组织如何与 ledger 交互的业务逻辑(例如，更改资产所有权的交易)包含在智能合约中。包含智能合约的结构称为链码( chaincode )，安装在相关的 *[ relevant ]* Peer 上，由相关的 Peer 组织批准，并提交到通道上。通过这种方式，您可以认为 chanicode 在物理上托管在 Peer 上，但在逻辑上托管在 Channel 上。在我们的例子中，链码 S5 被安装在每个 peer 上，即使组织不需要安装每个链码。请注意，ording service 没有安装链码，因为 ordering 节点通常不会提议交易。安装、批准和提交链代码的过程被称为链代码的“生命周期”。要了解更多信息，请查看  [Fabric chaincode lifecycle](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode_lifecycle.html)。

链码定义中提供的最重要的信息是背书策略 *[ [endorsement policy](https://hyperledger-fabric.readthedocs.io/en/latest/glossary.html#endorsement-policy) ]*。它描述了在交易被其他组织接受之前，哪些组织必须在他们的分类账副本上认可交易。根据用例，可以将背书策略设置为通道中的任何成员组合。如果没有设置背书策略，则从通道配置中指定的默认背书策略继承。

请注意，虽然一些链码包含了在通道上的成员之间创建私有数据事务  [private data transactions](https://hyperledger-fabric.readthedocs.io/en/latest/private_data_tutorial.html) 的能力，但私有数据超出了本主题的范围。

虽然现在技术上可以使用对等的 CLI 驱动事务，但最佳实践是创建一个应用程序，并使用它来调用链代码上的事务。

## Using an application on the channel

After a smart contract has been committed, client applications can be  used to invoke transactions on a chaincode, via the Fabric Gateway  service (the gateway). This completes the structure we showed in the  first image:

![xx](https://hyperledger-fabric.readthedocs.io/en/latest/_images/network.diagram.1.png)



Just like peers and orderers, a client application has an identity that  associates it with an organization. In our example, client application  A1 is associated with organization R1 and is connected to C1.

Starting in Fabric v2.4, the client application (developed using a  Gateway SDK v1.x) makes a gRPC connection to the gateway service, which  then handles the transaction proposal and endorsement process on behalf  of the application. The transaction proposal serves as input to the  chaincode, which uses it to generate a transaction response.

从 Fabric v2.4 开始，客户端应用程序(使用 Gateway SDK v1.x 开发) 使用 gRPC 连接到 gateway service，然后网关服务代表应用程序处理交易建议和背书过程。交易建议作为链码的输入，链码使用它来生成交易响应。

We can see that our peer organizations, R1 and R2, are fully  participating in the channel. Their applications can access the ledger  L1 via smart contract S5 to generate transactions that will be endorsed  by the organizations specified in the endorsement policy and written to  the ledger.

我们可以看到，我们的 peer 组织 R1 和 R2 都在充分参与到这个渠道中。他们的应用程序可以通过智能合约 S5 访问 Ledger L1，以生成由背书策略中指定的组织背书并写入分类账的交易。

Note: Fabric v2.3 SDKs embed the logic of the v2.4 Fabric Gateway service in the client application — refer to the [v2.3 Applications and Peers](https://hyperledger-fabric.readthedocs.io/en/release-2.3/peers/peers.html#applications-and-peers) topic for details.

For more information about how to develop an application, check out [Developing applications](https://hyperledger-fabric.readthedocs.io/en/latest/developapps/developing_applications.html).



## Joining components to multiple channels

现在，我们已经展示了如何创建通道的过程，以及组织、节点、策略、链码和应用程序之间的高层交互的性质，让我们通过向场景添加新组织和新通道来扩展我们的视图。为了展示 Fabric 组件如何连接到多个通道，我们将把 R2 及其对等体 P2 连接到新通道，而 R1 和 P1 不会被连接。

### Creating the new channel configuration

As we’ve seen, the first step in creating a channel is to create its  configuration. This channel will include not just R2 and R0, but a new  organization, R3, which has had its identities and certificates created  for it by CA3. R1 will have no rights over this channel and will not be  able to join components to it. In fact, it has no way to know it even  exists!

![network.5](https://hyperledger-fabric.readthedocs.io/en/latest/_images/network.diagram.5.png)

As before, now that the channel configuration, CC2, has been created, the channel can be said to **logically** exist, even though no components are joined to it.

So let’s join some components to it!



### Join components to the new channel

Just as we did with C1, let’s join our components to C2. Because we  already showed how all channels have a ledger and how chaincodes are  installed on peers and committed to a channel (in this case, the  chaincode is called S6), we’ll skip those steps for now to show the end  state of C2. Note that this channel has its own ledger, L2, which is  completely separate from the ledger of C1. That’s because even though R2 (and its peer, P2) are joined to both channels, the two channels are  entirely separate administrative domains.

![network.6](https://hyperledger-fabric.readthedocs.io/en/latest/_images/network.diagram.6.png)

Note that while both C1 and C2 both have the same orderer  organization joined to it, R0, different ordering nodes are servicing  each channel. This is not a mandatory configuration because even if the  same ordering nodes are joined to multiple channels, each channel has a  separate instance of the ordering service, and is more common in  channels in which multiple orderer organizations come together to  contribute nodes to an ordering service. Note that only the ordering  node joined to a particular channel has the ledger of that channel.

While it would also be possible for R2 to deploy a new peer to join to  channel C2, in this case they have chosen to deploy the P2 to C2. Note  that P2 has both the ledger of C1 (called L1) and the ledger of C2  (called L2) on its file system. Similarly, R2 has chosen to modify its  application, A2, to be able to be used with C2, while R3’s application,  A3, is being used with C2.

Logically, this is all very similar to the creation of C1. Two peer  organizations come together with an ordering organization to create a  channel and join components and a chaincode to it.

Think about this configuration from the standpoint of R2, which is  joined to both channels. From their perspective, they might think about  both C1 and C2, as well as the components they have joined to both, as  the “network”, even though both channels are distinct from each other.  In this sense, a “network” can also be seen as existing within the  perspective of a particular organization as “all of the channels I am a  member of and all of the components I own”.

Now that we have shown how organizations and their components can be  joined to multiple channels, let’s talk about how an organization and  its components are added to an existing channel.



## Adding an organization to an existing channel

As channels mature, it is natural that its configuration will also  mature, reflecting changes in the world that must be reflected in the  channel. One of the more common ways a channel will be modified is to  add new organizations to it. While it also possible to add more orderer  organizations (who may or may not contribute their own nodes), in this  example we’ll describe the process of how a peer organization, R3, is  added to the channel configuration CC1 of channel C1.

**Note that rights and permissions are defined at a channel  level. Just because an organization is an administrator of one channel  does not mean it will be an administrator of a different channel. Each  channel is a distinct administrative zone and fully customizable to the  use case it’s serving.**

![network.7](https://hyperledger-fabric.readthedocs.io/en/latest/_images/network.diagram.7.png)

Although the update to the diagram looks like one simple step, adding a new organization to a channel is, at a high level, a three step  process:

1. Decide on the new organization’s permissions and role. The full  scope of these rights must be agreed to before R3 is added to C1 and is  beyond the scope of this topic, but comprise the same kinds of questions that must be answered when creating a channel in the first place. What  kind of permissions and rights will R3 have on C1? Will it be an admin  on the channel? Will its access to any channel resources be restricted  (for example, R3 might only be able to write to C1, which means it can  propose changes but not sign them)? What chaincodes will R3 install on  its peers?
2. Update the channel, including the relevant chaincodes, to reflect these decisions.
3. The organization joins its peer nodes (and potentially ordering nodes) to the channel and begins participating.

In this topic, we’ll assume that R3 will join C1 with the same rights and status enjoyed by R1 and R2. Similarly, R3 will also be joined as  an endorser of the S5 chaincode, which means that R1 or R2 must redefine S5 (specifically, the endorsement policy section of the chaincode  definition) and approve it on the channel.

Updating the channel configuration creates a new configuration block, CC1.1, which will serve as the channel configuration until it is  updated again. Note that even though the configuration has changed, the  channel still exists and P1 and P2 are still joined to it. There is no  need to re-add organizations or peers to the channel.

For more information about the process of adding an organization to a channel, check out [Adding an org to a channel](https://hyperledger-fabric.readthedocs.io/en/latest/channel_update_tutorial.html).

For more information about policies (which define the roles organizations have on a channel), check out [Policies](https://hyperledger-fabric.readthedocs.io/en/latest/policies/policies.html).

For more information about upgrading a chaincode, check out [Upgrade a chaincode](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode_lifecycle.html#upgrade-a-chaincode).



### Adding existing components to the newly joined channel

Now that R3 is able to fully participate in channel C1, it can add  its components to the channel. Rather than do this one component at a  time, let’s show how its peer, its local copy of a ledger, a smart  contract and a client application can be joined all at once!

![network.8](https://hyperledger-fabric.readthedocs.io/en/latest/_images/network.diagram.8.png)

In this example, R3 adds P3, which was previously joined to C2, to  C1. When it does this, P3 pulls C1’s ledger, L1. As we mentioned in the  previous section, R3 has been added to C1 with equivalent rights as R1  and R2. Similarly, because the chaincode S5 was redefined and reapproved on the channel to include R3, R3 can now install S5 and begin  transacting. Just as R2 modified its application A2 to be able to be  used with channel C2, A3 is also now able to invoke transactions on C1.



## Network recap

We’ve covered a lot of ground in this topic. We’ve gone from a simple configuration with two organizations transacting on a single channel to multiple organizations transacting on multiple channels as well as the  process for joining an organization to a channel that already exists.

While this topic represents a relatively simple case, there are  endless combinations of sophisticated topologies which are possible to  achieve in Fabric, supporting an endless number of operational goals,  and no theoretical limit to how big a network can get. The careful use  of network and channel policies allow even large networks to be  well-governed.







照猫画虎的猫：

- [https://www.redhat.com/sysadmin/replacing-rclocal-systemd](https://www.redhat.com/sysadmin/replacing-rclocal-systemd)





