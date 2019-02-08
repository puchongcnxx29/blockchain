---

copyright:
  years: 2017, 2018
lastupdated: "2018-12-07"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# 词汇表
{: #glossary}

***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

本主题定义了本文档中出现的特定于 {{site.data.keyword.blockchainfull}} Platform 的术语。要更深入地了解术语并获取与 Hyperledger Fabric 概念相关的术语的词汇表，请参阅 [Hyperledger Fabric 词汇表 ![外部链接图标](images/external_link.svg "外部链接图标")](http://hyperledger-fabric.readthedocs.io/en/release-1.2/glossary.html)。
{:shortdesc}

## 安装 (Install)
在同级的文件系统上放置链代码的过程。您必须在将运行此链代码的每个同级上安装此链代码。

## 参与者 (Participant)
与区块链网络交互的任何组织、个人、应用程序或设备。在“参与者”伞下，有两个不同的分组：成员和用户。

## 成员 (Member) 
也称为“组织”，是区块链网络中的成员，类似于任何组的成员，用于构成网络结构。成员可以大至跨国公司，也可以小至个人。成员使用证书注册到网络中，证书授予成员将网络用作服务提供者（例如，签发证书、对事务处理进行验证/排序）或使用者的许可权。服务提供者提供基础的区块链服务，包括事务处理验证、事务处理排序和证书管理服务。使用者成员使用网络对分布式分类帐调用事务处理。成员可以有多个同级。

## 当前状态 (Current state)
分类帐的当前状态表示在其链事务日志中包含的所有键的最新值。因为当前状态表示通道已知的所有最新键值，有时候也称为“全局状态”。链代码针对当前状态数据执行事务处理建议。每次交换键值或添加新键时，当前状态都会发生更改。当前状态对于事务处理流程至关重要，因为最新键值对必须已知，然后才能对其进行更改。针对块中每个有效事务处理，同级都会将最新值落实到分类帐的当前状态。当前状态会存储在同级状态数据库中。

## 动态成员资格 (Dynamic membership)
成员可以由具有**登录者**特权的用户动态添加到网络。还会为成员分配角色和属性，以控制成员在网络上的访问权和权限。但角色和属性都不能动态分配。Hyperledger Fabric 支持添加或除去成员、同级和排序服务节点，而不会影响整个网络的运行。在业务关系调整并且出于各种原因需要添加或除去实体时，动态成员资格至关重要。

## 分类帐 (Ledger)
由一个用于存储不可变的有序事务处理记录的字面“区块链”以及一个用于维护当前状态的状态数据库组成。每个通道都有一个分类帐，并且对分类帐的更新根据特定通道的策略通过共识过程进行管理。

## 服务凭证 (Service credentials)
服务凭证采用 JSON 格式，并且包含针对网络资源（即 CA、排序者和同级）的 API 端点信息以及注册标识/密钥。应用程序通过这些 API 端点与网络资源进行交互。

## 共识 (Consensus)
一种协作流程，用于使分类帐事务处理在整个网络上保持同步。共识可确保仅当相应的参与者批准事务处理时才更新分类帐，并且会使用相同的事务处理按照相同的顺序来更新分类帐。有许多不同的算法可用于达成共识。

## 节点 (Node)
区块链的通信实体。有三种类型的节点：CA、同级和排序者。

## 客户机 (Client)
客户机表示代表用户行事的实体。客户机必须连接到同级才能与区块链进行通信。客户机可能会连接到自己选择的任何同级。客户机会创建事务处理，从而调用事务处理。客户机向支持者提交实际的事务处理调用，并将事务处理建议广播给排序服务。

## 连接概要文件 (Connection profile)
单击**连接概要文件**按钮时，连接概要文件会显示在“网络监视器”的“概述”屏幕中。这些信息以 JSON 格式提供，并包含网络资源（即同级、排序者和 CA）的 API 端点信息和 enrollID/secret。应用程序通过这些 API 端点与网络资源进行交互。

## 联盟 (Consortium)
在排序者系统通道上列出的非排序者组织的群组。这些是唯一可以创建通道的组织。在通道创建时，添加到通道的所有组织必须是联盟的一部分。但是，未在联盟中定义的组织可以添加到现有通道。虽然区块链网络可以有多个联盟，但大多数区块链网络都只有一个联盟。

## 链 (Chain)
分类帐的链是一种事务处理日志，构造为散列链接的事务处理区块。同级从排序服务接收事务处理区块，根据支持策略和并发违例将区块的事务处理标记为有效或无效，然后将该区块附加到同级文件系统上的散列链。

## 链代码 (Chaincode)
链代码也称为**智能合同**，是一些软件片段，其中包含对分类帐进行查询或更新的一组函数。

## 排序者 (Orderer)
该节点收集来自网络成员的事务处理，对事务处理排序，然后将其捆绑成区块。随后，这些区块分发给同级，同级在验证这些区块后，将其添加到每个通道上的分类帐中。排序者包含与每个成员绑定的加密身份资料，并对要访问网络的客户机和同级的身份进行认证。排序节点或节点集合提供的整体功能称为**排序服务**。

## 起源区块 (Genesis Block)
用于初始化区块链网络或通道的配置区块，也用作链中的第一个区块。

## 区块 (Block)
一组有序的事务处理，以加密方式链接到通道上的前一个区块。

## 全局状态 (World state)
请参阅[当前状态 (Current state)](#current-state)。

## 实例化 (Instantiate)
在特定通道上启动和初始化链代码容器的过程。链代码安装在同级上，并且每个同级都加入该通道后，必须在通道上对链代码进行实例化。实例化操作会对链代码执行任何必要的初始化，其中包括设置组成链代码初始全局状态的键值对。实例化之后，安装了链代码的同级即可以接受链代码调用。

## 事务处理 (Transaction)
区块链网络参与者用于与资产进行交互的机制。事务处理会创建新的链代码，或者调用现有链代码中的操作。

## 通道 (Channel)
由希望以专用方式处理事务的网络成员子集构成。通道通过允许通道成员建立特定规则和只有通道成员可以访问的单独分类帐，从而提供数据隔离和机密性。同级是充当组织的事务处理端点的节点，这些节点加入到通道。

## 同级 (Peer)
一种区块链网络资源，用于提供各种服务来执行和验证事务处理，并维护分类帐。同级会运行链代码，并且是网络通道上的事务处理历史记录和资产当前状态的持有者，即分类帐持有者。同级由组织拥有和管理，并加入到通道。

## 网络 (Network)
{{site.data.keyword.cloud_notm}} 上的 {{site.data.keyword.blockchainfull_notm}} Platform 服务实例。

## 网络监视器 (Network Monitor)
{{site.data.keyword.blockchainfull_notm}} Platform 提供的 GUI 仪表板，用于查看和管理区块链网络。

## 网络凭证 (Network Credentials)
可在“网络监视器”的“API”屏幕中查看。在 Swagger UI 中，凭证包含“key”（用户名）和“secret”（密码）。在试用 REST API 之前，您需要使用这些网络凭证进行认证。

## 业务网络 (Business network)
区块链网络定义，其中包含区块链解决方案的数据模型、事务处理逻辑和访问控制规则。“业务网络定义”是使用 [Hyperledger Composer](#hyperledger-composer) 创建的。业务网络定义会打包为可部署的 **.bna**（业务网络归档）文件。

## 用户 (User)
用户是区块链网络中通过与现有成员的信任关系而对分类帐具有间接访问权的参与者。

## 支持 (Endorsement)
链代码事务处理的验证过程。支持规则通过指定支持策略来实现。

## 支持策略 (Endorsement policy)
定义通道上必须执行附加到特定链代码应用程序的事务处理的同级节点，以及必需的响应（支持）组合。策略可能要求事务处理得到最少数量的支持同级、最低百分比的支持同级或分配给特定链代码应用程序的所有支持同级的支持。策略可以由支持同级根据应用程序以及针对不当行为（不管是否故意）所需的弹性级别进行组织。提交的事务处理必须满足支持策略后，提交同级才会将其标记为有效。此外还需要用于安装和实例化事务处理的明确支持策略。

## 智能合同 (smart contract)
请参阅[链代码](#chaincode)。

## 状态数据库 (State database)
当前状态数据会存储在同级上的数据库中，以便从链代码执行高效读取和查询。入门套餐网络将 CouchDB 用作状态数据库。企业套餐网络可以使用 LevelDB 或 CouchDB。

## 资产 (Asset)
有形或无形的商品、服务或财产，表示为在区块链网络上交易的物品。

## 组织 (Organization)
请参阅[成员 (Member)](#member)。

## CA
“认证中心”的缩写，这是向所有参与成员发放证书的组件。这些证书表示成员的身份。网络中的所有实体（同级、排序者、客户机等）必须具有通信、认证和最终进行事务处理的身份。任何直接参与区块链网络的情况都需要这些身份。您可以为 CA 创建业务网络卡。然后可以导入 [CA 卡](/docs/services/blockchain/develop_starter.html)，该卡用于将管理私钥交换为入门套餐认证中心的有效证书。

## CouchDB
在入门套餐网络中用于状态数据库的文档库。CouchDB 也是企业套餐网络的一个选项，另一个选项是 LevelDB。CouchDB 支持使用索引，并允许您对同级上的数据发出富文本查询。

## Gossip
Hyperledger Fabric 允许同级相互收集重要的网络信息，而不必依靠订购服务。[Gossip 数据传播协议 ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.1/gossip.html) 提供安全、可靠且可扩展的方式以供同级相互交换消息。例如，如果同级由于延迟、网络中断或其他原因而缺少一些区块，他们可使用 gossip 消息传递来联系拥有这些缺失区块的其他同级以同步为最新分类帐状态。

## HSM
硬件安全模块。将按需加密、密钥管理和密钥存储作为受管服务提供。HSM 是一种物理设备，可处理资源密集型加密处理任务并减少应用程序的等待时间。有关更多信息，请参阅[硬件安全模块 ![外部链接图标](images/external_link.svg "外部链接图标 ")](https://www.ibm.com/cloud/hardware-security-module)

## Hyperledger Composer
[Hyperledger Composer ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger.github.io/composer/latest/introduction/introduction.html) 是一种开放式源代码开发工具集。它使用定制建模语言，该语言与 JavaScript 事务处理和访问控制规则组合使用，对区块链业务网络进行完全建模。在将任何内容部署到真正的区块链之前，可以先使用 Hyperledger Composer 将现有系统和数据与区块链应用程序相集成。

## Hyperledger Fabric
[Hyperledger Fabric ![外部链接图标](images/external_link.svg "外部链接图标")](http://hyperledger-fabric.readthedocs.io/en/release-1.1/) 是 Linux Foundation 托管的一种业务区块链框架，充当通过模块化体系结构来开发区块链应用程序或解决方案的基础。Hyperledger Fabric 组件（如共识和成员资格服务）是即插即用的。

## Kafka
Hyperledger Fabric 的共识插件实现，用于在区块链网络中生成排序服务节点集群。Kafka 实现不适用于生产网络。

## LevelDB
键值存储库，是针对企业套餐网络的一个状态数据库选项，另一个选项是 CouchDB。LevelDB 将当前状态存储为键/值对，但不支持使用索引或富文本查询。

## MSP
成员资格服务提供者。一组加密机制和协议，用于在整个区块链网络中签发和验证证书与身份。在成员资格服务提供者作用域内发布的身份可以在该成员资格服务提供者的规则验证策略中进行求值。MSP 会安装在每个通道同级上，以确保向同级发出的事务处理请求源自已认证和已授权的用户身份。

## SDK
Hyperledger Fabric 支持两个软件开发包 (SDK)：Node SDK 和 Java SDK。Node SDK 可以通过 NPM 安装，Java SDK 可以通过 Maven 安装。SDK 有自己的 Git 存储库，即 [Fabric Node SDK ![外部链接图标](images/external_link.svg "外部链接图标")](https://github.com/hyperledger/fabric-sdk-node) 和 [Fabric Java SDK ![外部链接图标](images/external_link.svg "外部链接图标")](https://github.com/hyperledger/fabric-sdk-java)，并提供可用 API 的文档。Hyperledger Fabric 客户机 SDK 支持客户机应用程序与区块链网络之间的交互。

## SignCert
任何实体（无论组织还是管理员）附加到其建议或建议响应的证书。这些 signCert 对于实体是唯一的，并且由排序服务进行检查，以确保它们与该实体的文件中的 signCert 相匹配。

## SOLO
Hyperledger Fabric 的共识插件实现，用于在区块链网络中生成单个排序服务节点。入门套餐网络使用 SOLO 实现。SOLO 实现不适用于生产网络。SOLO 的替代方法是 Kafka 集群。
