---

copyright:
  years: 2017, 2018
lastupdated: "2018-11-27"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# 关于 {{site.data.keyword.blockchainfull_notm}} Platform for Amazon Web Services
{: #remote-peer-overview}

***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

**注：**{{site.data.keyword.blockchainfull}} Platform Remote Peer on ICP (Beta) 程序已终结。如果您仍希望在 ICP 环境中运行同级，请改为使用 **{{site.data.keyword.blockchainfull_notm}} Platform for ICP** 产品。有关更多信息，请参阅[关于 ICP 上的同级](/docs/services/blockchain/ibp-for-icp-about.html#ibp-icp-peer)。

在将 {{site.data.keyword.blockchainfull_notm}} Platform 同级连接到 {{site.data.keyword.cloud_notm}} 中的现有区块链网络后，可以在 AWS 云中运行该同级。在 {{site.data.keyword.cloud_notm}} 之外运行同级不但让扩展或加入区块链网络变得更加灵活，而且可利用 {{site.data.keyword.cloud_notm}} 中的现有网络。AWS 云中的同级利用 Platform 上的认证中心 (CA) 和排序服务，但是允许您将同级与 {{site.data.keyword.cloud_notm}} 之外的其他应用程序并置在一起。
{:shortdesc}


## 注意事项
{: #remote-peer-limitations}

{{site.data.keyword.blockchainfull_notm}} Platform for AWS 无权访问在 {{site.data.keyword.blockchainfull_notm}} Platform 上托管的同级的完整功能，也无权获得完全支持。在运行 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 之前，请确保了解以下限制和局限性：
- 在其他云环境中运行的同级在 {{site.data.keyword.cloud_notm}} 上的区块链网络的“网络监视器”中不可见。
- 无法使用“网络监视器 UI”中的 Swagger UI 寻址到在 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 上运行的同级。
- 您负责管理 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 同级节点的运行状况监视、安全性、日志记录和资源使用情况。
- 只能将 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 同级连接到 Fabric V1.1 或 V1.2.1 级别的区块链网络。可以通过打开“网络监视器”中的[“网络首选项”窗口](/docs/services/blockchain/v10_dashboard.html#network-preferences)来找到 Fabric 版本。
- {{site.data.keyword.blockchainfull_notm}} Platform for AWS 同级的数据库类型必须匹配区块链网络的数据库类型，即 LevelDB 或 CouchDB。
- CouchDB Fauxton 接口在 AWS 同级上不可用。
- 目前不支持将 [Gossip](../glossary.html#gossip) 用于 AWS 同级。这意味着也不支持依赖于 Gossip 的功能，例如[专用数据 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/private-data-arch.html "private data") 和[服务发现 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/discovery-overview.html "service discovery")。

## 先决条件
{: #remote-peer-prereq}

要使用 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 同级，您必须具有属于 {{site.data.keyword.blockchainfull_notm}} Platform 上的入门套餐或企业套餐网络的组织。{{site.data.keyword.blockchainfull_notm}} Platform for AWS 同级利用 {{site.data.keyword.blockchainfull_notm}} Platform 网络的 API 端点、Hyperledger Fabric CA 和排序服务进行操作。如果您不是任何区块链网络的成员，那么需要创建或加入一个网络。有关更多信息，请参阅[创建网络](/docs/services/blockchain/get_start.html#creating-a-network)或[加入网络](/docs/services/blockchain/get_start.html#joining-a-network)。

## 许可和定价
{: #remote-peer-license-pricing}

目前，{{site.data.keyword.blockchainfull_notm}} Platform for AWS 作为 Community Edition 免费提供；未来，IBP for AWS 可能会更改为自带许可证 (BYOL) 模型，该模型将要求向 IBM 购买许可证。

**注：**要操作 AWS 同级，您必须具有属于 {{site.data.keyword.blockchainfull_notm}} Platform 上的入门套餐或企业套餐网络的组织。这意味着您或网络中的其他成员必须为组织支付 {{site.data.keyword.blockchainfull_notm}} [成员资格费用](/docs/services/blockchain/pricing.html#key-elements-of-pricing)。有关支付费用的更多信息，请参阅[付费方式](/docs/services/blockchain/paying_mode.html)。


## 部署 AWS 同级
{: #deploy-remote-peer}

使用 AWS 的[快速入门模板 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://aws.amazon.com/quickstart/architecture/ibm-blockchain-platform/ "快速入门模板") 可轻松部署 {{site.data.keyword.blockchainfull_notm}} Platform for AWS。有关更多信息，请参阅 [{{site.data.keyword.blockchainfull_notm}} Platform for AWS Quick Start Deployment Guide ![外部链接图标](../images/external_link.svg "外部链接图标")](https://s3.amazonaws.com/aws-quickstart/quickstart-ibm-fabric/doc/ibm-blockchain-platform-for-aws.pdf "IBM Blockchain Platform for AWS Quick Start Reference Deployment")。

有关如何部署 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 的指示信息，请参阅[在 Amazon Web Services 中部署同级](/docs/services/blockchain/remote_peer_aws.html "在 Amazon Web Services 中部署同级")。

下图描述了部署 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 同级的过程。

<img usemap="#home_map1" border="0" class="image" id="image_ztx_crb_f1b2" src="../images/remote_peer_AWS_flow.png" width="750" alt="单击一个框可获取有关过程的更多详细信息。" style="width:750px;" />
<map name="home_map1" id="home_map1">
<area href="remote_peer_aws.html#remote-peer-aws-account" alt="配置或访问 AWS" title="配置或访问" shape="rect" coords="157.05, 52.53, 283.62, 127.11" />
<area href="remote_peer_aws.html#remote-peer-aws-account" alt="创建密钥对" title="创建密钥对" shape="rect" coords="300.97, 52.53, 427.54, 127.11" />
<area href="remote_peer_aws.html#prerequisites-aws" alt="创建或加入网络" title="创建或加入网络" shape="rect" coords="157.05, 131.8, 283.62, 206.37" />
<area href="remote_peer_operate_aws.html#aws-peer-operate-with-sdk" alt="加入通道" title="加入通道" shape="rect" coords="300.97, 131.8, 427.54, 206.37" />
<area href="remote_peer_aws.html#aws-register-peer" alt="注册同级身份" title="注册同级身份" shape="rect" coords="443.95, 131.8, 570.53, 206.37" />
<area href="remote_peer_aws.html#aws-network-endpoints" alt="检索同级配置信息" title="检索同级配置信息" shape="rect" coords="585.53, 131.8, 712.1, 206.37" />
<area href="remote_peer_aws.html#remote-peer-aws-launchqs" alt="单击链接" title="单击链接" shape="rect" coords="157.05, 258.43, 283.62, 333.48" />
<area href="remote_peer_aws.html#remote-peer-aws-launchqs" alt="配置同级实例" title="配置同级实例" shape="rect" coords="300.97, 258.43, 427.54, 333.48" />
<area href="remote_peer_aws.html#remote-peer-aws-test" alt="验证部署" title="验证部署" shape="rect" coords="443.95, 258.43, 570.53, 333.48" />
<area href="remote_peer_operate_aws.html#aws-peer-operate-with-sdk" alt="使用 Fabric SDK" title="使用 Fabric SDK" shape="rect" coords="157.05, 338.64, 283.62, 413" />
<area href="remote_peer_operate_aws.html#aws-peer-cli-operate" alt="使用 Fabric 工具 CLI" title="使用 Fabric 工具 CLI" shape="rect" coords="443.95, 338.64, 570.53, 413" />
</map>

*图 1. AWS 上的 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 部署流程*


## 操作 AWS 同级
{: #operate-remote-peer}

部署 AWS 同级后，需要完成若干操作步骤，然后同级才能向网络提交事务。这些操作步骤包括将组织添加到通道，将同级加入通道，在同级上安装链代码，在通道上实例化链代码，以及将应用程序连接到同级。有关更多信息，请参阅[在 Amazon Web Services 中操作同级](/docs/services/blockchain/remote_peer_operate_aws.html#remote-peer-operate-aws)。

## 数据存储位置
{: #data-residency}

由于区块链网络并不关注所处理的数据类型，因此有时必须采取额外的步骤来确保某些类型数据的安全。数据存储位置的最常见需求与特定国家或地区的法律相关，这要求在 IT 系统中处理和存储的所有数据必须保留在特定国家或地区境内。与此类似，受到高度监管的行业（例如，政府、医疗保健和金融服务）中的某些公司要求必须将数据完全存储在其防火墙之后。因此，为实现数据存储位置，区块链网络的所有组件都必须属于相同[通道](/docs/services/blockchain/glossary.html#channel)并且位于单个国家或地区境内。

为满足数据存储位置需求，重要的是了解作为 {{site.data.keyword.blockchainfull_notm}} Platform 基础的 Hyperledger Fabric 体系结构。该体系结构主要围绕着三个关键组件：认证中心 (CA)、排序节点和同级。同级以区块形式接收来自排序服务的已排序状态更新并维护状态和分类帐。因此，同级和排序节点具有直接关系。分类帐包含所有事务日志所包含的所有密钥和数据的最新值。

此外，客户机应用程序使用 [Fabric SDK](./docs/services/blockchain/v10_application.html#using-the-fabric-sdks) 以将事务发送给同级和排序服务。这些事务包含[读/写集语义 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/readwrite.html "读/写集语义") 数据，其中包含有关分类帐的密钥/值对。

如果国家或地区内数据存储位置是您企业的需求，那么排序节点、同级和客户机应用程序必须位于同一国家或地区内。在 {{site.data.keyword.cloud_notm}} 中创建 {{site.data.keyword.blockchainfull_notm}} Platform 网络时，您可以选择网络的位置。<!--For a Starter Plan network, you can select from US South, United Kingdom, and Sydney. For an Enterprise Plan network, you can select from currently available locations, which include Dallas, Frankfurt, London, Sao Paulo, Tokyo, and Toronto. -->有关区域和位置的更多信息，请参阅 [{{site.data.keyword.blockchainfull_notm}} Platform 区域和位置](/docs/services/blockchain/{site.data.keyword.blockchainfull_notm}} Platform 网络位置相同的国家或地区中。

### 数据存储位置的用例

假设 {{site.data.keyword.blockchainfull_notm}} Platform 网络包含排序节点和认证中心，并且有包含四个组织的联盟。这些组织都有一个或多个同级节点。所有四个组织都属于一个通道，网络的所有组件都位于部署了 {{site.data.keyword.blockchainfull_notm}} Platform 网络的区域（例如，法兰克福）。最后，与同级进行交互的客户机应用程序也位于德国境内。保持了数据存储位置。  

![所有组件位于同一国家或地区时的数据存储位置](../images/remote_peer_data_res_1.png "所有组件位于同一国家或地区时的数据存储位置")  
*图 3. 所有组件位于同一国家或地区时的数据存储位置*

现在，让我们来看看**同级**加入其中一个组织时会产生的影响。同级可以位于网络的其余部分所在的相同区域中，也可以位于 IBP 网络区域以外的任何位置：

-	如果同级位于网络的其余部分所在的相同国家或地区，那么保持了数据存储位置。所有分类帐数据都会保留在德国境内，如上面**图 3** 所示。
-	如果同级位于其他国家或地区（例如，美国），那么数据存储位置不再得到保持，因为同级分类帐上的数据在该国家或地区境外共享。

要解决此问题，可以使用**通道**将数据隔离到网络上的一部分同级。{{site.data.keyword.blockchainfull_notm}} Platform 网络包含跨越国家或地区边境的同级和排序节点时，通道可用于将分类帐数据与具有该国家或地区境外的同级的组织相隔离。  

**注：**排序节点总是位于您选择用于托管网络的数据中心区域内。不能有多个排序节点跨越国家或地区边境。但是，同级可位于数据中心内或位于 {{site.data.keyword.cloud_notm}} 之外的远程位置。

![同级位于 IBM Blockchain Platform 区域所在国家或地区之外时的数据存储位置](../images/remote_peer_data_res_2.png "同级位于 IBM Blockchain Platform 区域所在国家或地区之外时的数据存储位置")  
*图 4. 同级位于 IBM Blockchain Platform 区域所在国家或地区之外时的数据存储位置*

在**图 4** 中，`OrgC` 和 `OrgD` 不需要数据存储位置。事实上，`OrgD` 现在包含两个位于*美国*的同级 `OrgD-peer1` 和 `OrgD-peer2`。因此，为了使 `OrgA`、`OrgB` 及其各自的客户机应用程序和位于德国的同级能够隔离通道 `X` 上的分类帐数据，针对 `OrgC` 和 `OrgD` 创建了新的通道 `Y`。

要更深刻地了解 {{site.data.keyword.blockchainfull_notm}} Platform 网络上的数据流，请参阅[有关事务流的 Fabric 文档 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/txflow.html "事务流")。

将来，Hyperledger Fabric 中的新技术将使用[专用数据集合 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/private-data/private-data.html "专用数据集合") 和 Zero Knowledge Proof 来提高进一步实现数据存储位置的能力。

- 专用数据集合确保专用数据在同级之间共享（通过 Gossip 协议）时，只有授权的同级（例如，国家或地区境内的同级）才能看到该数据。数据存储在同级的专用数据库中。此处不涉及排序服务，并且排序服务也不会看到专用数据。会将散列化的此数据写入到通道上每个同级的分类帐。用于状态验证的散列充当事务证据，并且可用于审计目的。专用数据可用于 {{site.data.keyword.blockchainfull_notm}} Platform 上在 Fabric V1.2.1 上运行的网络。但是，专用数据功能对于同级不可用。

- Zero-Knowledge Proof (ZKP) 允许“证明者”向“验证者”保证他们在不泄露自身秘密的情况下了解保密信息。这是显示您知道某些项满足条件而不显示您所知道的内容的方法。

您可以在有关 [Private and confidential transactions with Hyperledger Fabric ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/developerworks/cloud/library/cl-blockchain-private-confidential-transactions-hyperledger-fabric-zero-knowledge-proof/index.html "Private and confidential transactions with Hyperledger Fabric") 的白皮书中了解这些技术的更多相关信息。

## 获取支持
{: #remote-peer-support}

IBM Blockchain Platform 不提供对此产品的支持。如果遇到与同级相关的问题，您可以使用免费区块链开发人员资源和支持论坛，并从 {{site.data.keyword.IBM_notm}} 和 Fabric 社区获取帮助。有关更多信息，请参阅[区块链资源和支持论坛](/docs/services/blockchain/ibmblockchain_support.html#resources)。您还可以在“网络监视器”的**获取帮助**屏幕上查看支持资源。

- 对于与 AWS 相关的问题，您可以使用[社区支持论坛 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://forums.aws.amazon.com/index.jspa "AWS 社区支持论坛") 和 [AWS 高端支持 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://aws.amazon.com/premiumsupport/ "AWS 高端支持")。

{{site.data.keyword.blockchainfull_notm}} 不支持在 {{site.data.keyword.cloud_notm}} 中开具并且与 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 相关的用例。Community Edition 用于探索、开发和测试，请勿将其用于生产。
