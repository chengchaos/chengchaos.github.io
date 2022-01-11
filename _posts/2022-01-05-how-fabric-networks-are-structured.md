---
title: How Fabric networks are structured
key: 2022-01-05
tags: Fabric fabric networks
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

he first step in creating a network or a channel is to agree to and then define its configuration:

![creaating the network](https://hyperledger-fabric.readthedocs.io/en/latest/_images/network.diagram.2.png)

channel 配置 CC1 已经得到组织 R1、R2 和 R0 的同意，并且包含在一个称为“配置块”的块中，该块通常是由 `configtxgen` 工具从 `configtx.yaml` 创建的。虽然一个组织可以单方面创建一个 channel ，然后邀请其他组织加入(我们将在《将一个组织添加到现有的渠道中》进行探讨)，但现在我们假设这些组织从一开始就希望在该渠道上进行合作。

一旦配置块存在，通道就可以说是**逻辑上存在的**，即使没有任何组件在物理上连接到它。这个配置块包含可以连接组件并在通道上交互的组织的记录，以及定义如何做出决策和达成特定结果的结构的**策略**。虽然 peers 和应用程序是网络中的关键角色，但它们在通道中的行为更多地由通道配置策略决定，而不是其他任何因素。有关策略以及它们如何在通道配置中定义的更多信息，请参阅 [Policies](https://hyperledger-fabric.readthedocs.io/en/latest/policies/policies.html).

这些组织的定义及其管理员的身份必须由与每个组织关联的证书颁发机构(CA)创建。在我们的示例中，组织 R1、R2 和 R0 的认证和组织定义分别由 CA1、CA2 和 CA0 创建。有关如何创建CA的信息，请参阅“为CA规划”。创建CA后，请参阅“向CA注册和注册身份”，以了解如何定义组织和为管理员和节点创建身份。

For more information about using `configtxgen` to create a configuration block, check out [Using configtx.yaml to build a channel configuration](https://hyperledger-fabric.readthedocs.io/en/latest/create_channel/create_channel_config.html).























照猫画虎的猫：

- [https://www.redhat.com/sysadmin/replacing-rclocal-systemd](https://www.redhat.com/sysadmin/replacing-rclocal-systemd)





