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

# {{site.data.keyword.blockchainfull_notm}} Platform

***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

**注意：**在使用 {{site.data.keyword.blockchainfull}} Platform 产品之前，请阅读[免责声明](/docs/services/blockchain/needtoknow.html)一节中的技术和支持信息。
{:shortdesc}

{{site.data.keyword.blockchainfull_notm}} Platform (IBP) 是唯一能够处理多组织区块链网络的完整生命周期的集成业务就绪型平台。它旨在通过每个阶段的协作来加速创建“为业务构建”全球区块链网络，该网络具有适用于最严苛用例和受监管行业的性能和安全性。{{site.data.keyword.cloud_notm}} 上的 {{site.data.keyword.IBM_notm}} 受管网络产品适合于区块链新客户。具备 Hyperledger Fabric 使用经验的客户可以通过 {{site.data.keyword.cloud_notm}} Private (ICP) 或 Amazon Web Services (AWS) 在其自己的基础架构上部署网络组件或网络。

{{site.data.keyword.blockchainfull_notm}} Platform 提供不同的产品，帮助所有类型的用户开启其区块链之旅并将其应用程序移动到生产环境。

|       |[**入门套餐**](#starter-enterprise)|[ **企业套餐**](#starter-enterprise)| [**IBP for ICP**](#ibp-for-icp) | [**IBP for AWS**](#ibp-for-aws)|
| ------------------------- |--------------------------|-----|-----|------|
| **包含的内容** | **基本服务级别、开发和测试环境** |**高级服务级别和企业生产就绪环境**|**可部署 CA、排序节点和同级 Helm 图表**|**AWS 同级快速入门**|
| **计费策略** | **每月预订与[可用云信用值](/docs/services/blockchain/howto/pricing.html#starter-plan-pricing)** |**每月预订**|  **[VPC 定价](/docs/services/blockchain/ibp-for-icp-about.html#ibp-icp-pricing)和免费 Community Edition**| **免费** |
| **云平台**| **IBM Cloud**|**IBM Cloud**|**IBM Cloud Private**| **AWS**|

**注意：**请勿将**入门套餐**用于生产用途。入门套餐是一个开发与测试环境，不适合生产工作负载。

## {{site.data.keyword.blockchainfull_notm}} Platform 功能

{{site.data.keyword.blockchainfull_notm}} 产品基于 [Hyperledger Fabric](/docs/services/blockchain/reference/v10_fabric.html) 代码库构建，该代码库利用模块化体系结构来实现企业级别的安全性、数据完整性、可扩展性和性能，以满足您的业务需要。

{{site.data.keyword.blockchainfull_notm}} Platform 提供高度安全和许可的区块链网络，通过此网络，已认证的成员可轻松定义资产并创建用于修改和交换资产的业务解决方案。借助
{{site.data.keyword.blockchainfull_notm}} Platform 产品，您可以利用编排框架**快速地将联盟组织到活动区块链网络**。{{site.data.keyword.blockchainfull_notm}}
Platform 提供了友好工具，旨在使多个机构易于彼此连接并创建以民主方式管理的网络。通过内置的仪表板监视器和供应的实用程序，网络的创建、管理和操作任务变得直观且透明。通过前文所述创建网络和实施管控不再是繁琐的流程，联盟还可以转而**专注于部署智能合同和转移资产和信息**。

网络不可或缺资源（同级、排序节点、认证中心）的**高可用性**消除了单点故障可能引起的严重影响。内置仪表板监视器支持轻松管理这些资源，同时提供强大的机制对资产和智能合同进行可视化。*仅在入门级和企业级产品上受支持*。

Hyperledger Fabric 体系结构的**模块性**和网络角色的显著分离提供了一个基础架构，用于为各种用例实现可伸缩性和适应能力。*在所有 {{site.data.keyword.blockchainfull_notm}} Platform 产品上均可用和受支持*。

在事务处理的整个生命周期内发挥作用的相互制衡机制可确保产生全面审查的一致结果，并且分类帐可通过实现 Gossip 协议来时刻保持同步。通过在整个网络不断地施行**签名/验证**操作，轻松地实施身份和访问控制（在所有* {{site.data.keyword.blockchainfull_notm}} Platform 产品上提供和受支持）*。

提供**管控工具**，允许成员为其网络管理重要业务规则。例如，您可能想要实施一个策略，用于定义网络中必须有多少成员同意才能允许新成员加入。或者，可能有一项资产需要每个参与者保证，才能进行修改。对于任何类型的业务网络来说，管控规则是必要的基础组成部分，通常它们会非常复杂。管控工具（如策略编辑器）可极大地简化此过程。*仅在入门级和企业级产品上提供*。

企业套餐网络在**高度安全和隔离**的环境中运行，没有对网络资源的外部访问权（包括根访问权）。数据在运行时和暂停时加密，硬件安全模块允许按照行业规定保护数字密钥。硬件虚拟化用于在隔离环境中运行每个节点，从而保护网络中的其他节点不受可能具有错误或恶意链代码的同级的影响。通过高级加密实现，加快了散列、签名/验证操作和节点对节点通信。*仅适用于企业*。

有关所有 Hyperledger Fabric 功能部件和功能的更多详细信息，请参阅 [Hyperledger Fabric 文档 ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/latest/ "Hyperledger Fabric 文档")。

**图 1** 描述了已部署的区块链网络的示例，其中包含四个成员（每个成员拥有两个同级）、负责分发密码身份资料的认证中心，以及一个定义策略和网络参与者的排序服务。蓝色通道包含全部四个网络成员，而黄色通道仅限于三个成员：银行 B、C 和 D。我们也看到银行 A 扮演了网络发起者的角色，银行 D 仅作为成员在黄色通道的上下文中存在。最后，拥有正确签名的 x509 证书的用户或应用程序能够向网络中的同级发送调用。

![区块链网络](images/blockchain_network_2-01.png "示例区块链网络")

*图 1. 由四个成员组成并利用通道隔离数据的区块链网络示例*

## 开发要在网络上运行的应用程序

利用从 400 多个客户合作项目所提炼出的最佳实践，探索和加速区块链的发展：

* 确保在拥有显著缩短区块链应用程序开发时间（原先需要六周时间开发的应用程序不到两天就能创建出来）的技术的企业之间保持紧密的一致性。
* 利用 {{site.data.keyword.blockchainfull_notm}} 开发者工具，在现有程序员名册中快速构建区块链技能。
* 为开发人员提供足够的灵活性，可使用开放式和现代工具集，在其首选环境中进行学习和开发。

作为企业所有者，您可以利用 IBM 实力雄厚的行业和区块链专家队伍来帮助开发用例，这些专家齐聚 {{site.data.keyword.blockchainfull_notm}} Garage，以充分发挥 {{site.data.keyword.blockchainfull_notm}} Platform 的全部能力。

作为开发者，您可以在 {{site.data.keyword.blockchainfull_notm}} Platform 网络环境中使用交互式环境来开发、迭代和测试业务网络，从而快速轻松满足业务需求并加快区块链应用程序开发。这些工具旨在将业务规则转变为您首选环境中的业务网络代码：

* **在线探索**
  利用 [{{site.data.keyword.blockchainfull_notm}} Platform: Develop](/docs/services/blockchain/develop.html)<!--, which is powered by an open source development tool--> 了解关键区块链概念，创建网络定义，并利用可复用的行业模型和智能合同库。

  开发了业务网络后，可以将其部署到在 {{site.data.keyword.blockchainfull_notm}} Platform 上运行的实时网络。有关更多信息，请参阅[使用入门套餐部署业务网络](/docs/services/blockchain/develop_starter.html)和[使用企业套餐部署业务网络](/docs/services/blockchain/develop_enterprise.html)。

* **本地安装**
  利用 Hyperledger Fabric 在您的笔记本电脑上直接开发和测试。有关更多信息，请参阅[构建第一个网络](http://hyperledger-fabric.readthedocs.io/en/release-1.2/build_network.html)。

* **在云环境中协作**
   通过入门套餐和企业套餐选项，使用随时可用型实时网络来开发代码并与他人共享。有关更多信息，请参阅[关于入门套餐](/docs/services/blockchain/starter_plan.html)和[关于企业套餐](/docs/services/blockchain/enterprise_plan.html)。

## {{site.data.keyword.IBM_notm}} 支持

{{site.data.keyword.IBM_notm}} 提供有关 {{site.data.keyword.IBM_notm}} 实施的区块链解决方案的各种支持选项。有关 {{site.data.keyword.blockchainfull_notm}} 支持的更多信息，请参阅[获取支持](/docs/services/blockchain/ibmblockchain_support.html)。


## 入门套餐和企业套餐
{: #starter-enterprise}

入门和企业套餐利用生产就绪型安全服务来支持您部署和操作分散的区块链网络。**入门套餐**是用于了解或开始开发区块链网络的环境，**企业套餐**是用于提供高级别安全性和支持的生产环境。开始很小，但通过利用以下功能，随着您成员资格和事务处理量的增加，弹性地对您的网络进行调整：

* 具有许多硬件、固件和软件安全功能的超高安全性环境。
* 强化的体系结构，实现可扩展性、弹性和可用性。
* 针对性能进行优化，并在全球最快的 Linux 计算上运行。

在 {{site.data.keyword.blockchainfull_notm}} Platform 上操作网络包括用于简化管理任务的工具和功能：

* 用于在网络上监视和管理资源的仪表板。
* 完整代码堆栈无缝升级。
* 集成到门户网站的全天候技术支持。
* 强化的安全堆栈，具有无特权访问、恶意软件和篡改抵制、100% 加密，以及许多其他功能，用于具有受监管行业中敏感数据的网络。

入门套餐和企业套餐是 {{site.data.keyword.cloud_notm}} 上的平台服务，两个套餐都遵循服务级别协议 (SLA) 上的 [{{site.data.keyword.cloud_notm}} 服务条款 ![外部链接图标](images/external_link.svg "外部链接图标")](http://www-03.ibm.com/software/sla/sladb.nsf/sla/bm "{{site.data.keyword.cloud_notm}} 服务条款")。请注意，入门套餐和企业套餐网络是跨**多个环境**进行供应的，这些环境分布在不同地理位置的多个数据中心。

有关入门套餐和企业套餐的更多信息，请参阅[关于入门套餐](starter_plan.html)和[关于企业套餐](enterprise_plan.html)。如果您准备开始使用，请立即在 {{site.data.keyword.cloud_notm}} 上注册 [IBP 入门或企业成员资格 ![外部链接图标](images/external_link.svg "外部链接图标")](https://console.bluemix.net/catalog/services/blockchain?env_id=ibm:yp:us-south&taxonomyNavigation=apps)！

## {{site.data.keyword.blockchainfull_notm}} Platform for ICP
{: #ibp-for-icp}

{{site.data.keyword.blockchainfull_notm}} Platform for ICP 提供了通过 ICP 在您自己的基础架构上运行区块链网络所需的组件。组件包括 Hyperledger Fabric、一个认证中心 (CA)、一个排序节点和一个同级，您可以使用 Kubernetes Helm 图表来部署、管理和设置。**本产品面向具备高级 Hyperledger Fabric 体验的客户。**

IBP for ICP 支持在私有云上部署区块链网络，以应对数据存储需求、市场监管和基础架构首选项。它通过 {{site.data.keyword.cloud_notm}} Private（基于 Kubernetes 的应用程序平台，用于开发和管理本地、容器化应用程序），简化在您自己的基础架构中部署区块链网络的基本元素的过程。

 * 使客户能够使用自己的基础架构来管理 IBP 网络。免费 Community Edition 允许客户在自己的隔离安全环境中运行，但不会提供任何支持。
 * 使客户能够通过使用 Hem 图表和详细的操作文档来配置 Fabric on Kubbernetes。
 * 使用高级技术支持来授权客户，除非您使用 Community Edition。

 有关 IBP for ICP 的更多信息，请参阅[关于 {{site.data.keyword.blockchainfull_notm}} Platform for {{site.data.keyword.cloud_notm}} Private](/docs/services/blockchain/ibp-for-icp-about.html)。

## {{site.data.keyword.blockchainfull_notm}} Platform for AWS
{: #ibp-for-aws}

{{site.data.keyword.blockchainfull_notm}} Platform for AWS 支持同级利用连接概要文件、Hyperledger Fabric 认证中心 (CA) 以及
{{site.data.keyword.cloud_notm}} 上现有入门套餐或企业套餐网络的排序服务，以通过 AWS 快速入门模板处理事务。快速入门允许您使用 AWS CloudFormation 模板部署同级。如果 IT 基础架构决策制定者和系统管理员希望快速配置、部署和运行 AWS 托管的 IBP 同级，并且这些同级连接到入门套餐或企业套餐网络，那么此模板非常适合。您可以使用该模板在 AWS 中构建新的虚拟专用云 (VPC)，或者将同级部署到现有 VPC 中。

快速入门将完成以下配置：
 * 跨两个可用性区域的高可用性体系结构。
 * 根据 AWS 最佳实践使用公共子网配置的 VPC。这将为您在 AWS 上提供您自己的虚拟网络。
 * 允许访问互联网的互联网网关。
 * 在公共子网中，两个可用性区域中的两个同级（每个子网中一个同级）。
 * 在每个公共子网中，具有嵌入式 LevelDB 数据库或辅助 CouchDB 容器的同级容器。

有关 IBP for AWS 的更多信息，请参阅[关于 {{site.data.keyword.blockchainfull_notm}} Platform for Amazon Web Services](/docs/services/blockchain/howto/remote_peer.html)。
