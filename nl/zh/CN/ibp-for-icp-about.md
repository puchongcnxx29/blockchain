---

copyright:
  years: 2018
lastupdated: "2018-12-07"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# 关于 {{site.data.keyword.blockchainfull_notm}} Platform for {{site.data.keyword.cloud_notm}} Private
{: #overview}

***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

{{site.data.keyword.blockchainfull}} Platform 发布了 IBM Blockchain Platform for {{site.data.keyword.cloud_notm}} Private(ICP)；这是一个应用程序平台，用于开发和管理基于 Kubernetes 的容器化应用程序，支持用户在 x86、LinuxONE 和 IBM Z 上部署认证中心 (CA)、排序节点和同级。{{site.data.keyword.blockchainfull_notm}} Platform for ICP 基于 Hyperledger Fabric V1.2.1，使用 Kubernetes Helm 图表进行部署。
{:shortdesc}

{{site.data.keyword.blockchainfull_notm}} Platform for {{site.data.keyword.cloud_notm}} Private 是一个捆绑产品，供 ICP 客户在其本地环境中部署区块链组件。导入 Helm 图表之后，您可在 ICP 目录中发现该项显示为 {{site.data.keyword.blockchainfull_notm}} Platform 磁贴。有关 ICP 的更多信息，请参阅 [{{site.data.keyword.cloud_notm}} Private V3.1.0 ![外部链接图标](images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/kc_welcome_containers.html "{{site.data.keyword.cloud_notm}} Private 3.1.0") 的文档。

## {{site.data.keyword.blockchainfull_notm}} Platform for ICP 提供的功能

通过 {{site.data.keyword.blockchainfull_notm}} Platform for IBM Cloud Private (ICP)，可以部署 Hyperledger Fabric 区块链的所有基本组件：认证中心、排序服务和同级。利用 ICP 可根据业务需求灵活地部署不同的组件。可以使用 {{site.data.keyword.blockchainfull_notm}} for ICP 通过部署和配置在区块链联盟中将组织绑定在一起的排序服务来启动区块链网络。此外，还可部署同级并加入其他使用基于 Fabric 的组件的网络，包括使用 ICP 跨云部署的 {{site.data.keyword.blockchainfull_notm}} Platform 网络，或在 IBM Cloud 上托管的入门套餐和企业套餐网络。有关 Hyperledger Fabric 网络构建块的更多信息，请参阅[区块链组件概述](/docs/services/blockchain/blockchain_component_overview.html)。

## {{site.data.keyword.blockchainfull_notm}} Platform for ICP 是否适用于您

通过在 {{site.data.keyword.cloud_notm}} 之外运行 {{site.data.keyword.blockchainfull_notm}} Platform 组件，可以更灵活地扩展或加入区块链网络。这也有助于网络创始人允许新成员在使用其所选平台时加入网络，从而扩展其网络。此外，还支持有兴趣加入区块链网络的组织使用其现有应用程序与其同级并置，或者与记录系统集成。

**重要信息：**部署 {{site.data.keyword.blockchainfull_notm}} Platform for ICP 的过程非常复杂，并且需要您掌握高水平的 Fabric 专业知识。如果您不熟悉 Fabric、ICP 或 {{site.data.keyword.blockchainfull_notm}} Platform，并且目标是设置开发环境或概念验证，请考虑改为使用[入门套餐](/docs/services/blockchain/starter_plan.html)。另请注意，{{site.data.keyword.blockchainfull_notm}} Platform for ICP 中不一定支持所有潜在的部署配置。

此产品的用户应负责管理其自己的安全性和基础架构。{{site.data.keyword.cloud_notm}} 不提供这些服务。开始之前，请查看下一部分中的[注意事项和限制](#ibp-icp-considerations)。

## 注意事项和限制
{: #ibp-icp-considerations}

开始之前，请确保您了解以下**注意事项**和**限制**：

- 您负责组件的运行状况监视、安全性、日志记录以及组件的资源使用情况管理。
- 在其他云环境中运行的组件在 {{site.data.keyword.cloud_notm}} 上运行的网络的“网络监视器”中不可见。
- Helm 图表部署排序节点、同级或 CA 的一个实例。
- 可以将多个组件部署到 ICP 中的一个名称空间，只要这些组件具有不同的发布名称即可。
- 无法使用“网络监视器 UI”中的 Swagger UI 寻址到组件。
- 不支持双向 TLS。

**CA 注意事项**
- 此 Helm 图表部署 CA 的一个实例。因为每个组织具有一个不同的 CA 被视为最佳实践，所以可能有必要部署多个 CA。例如，如果计划部署一个排序节点和三个同级，那么至少需要两个 CA（一个用于排序节点组织，另一个用于同级组织）。
- 虽然您可能会选择运行单独的 MySQL 数据库，但 Helm 图表中不存在此选项。不过，Helm 图表会在 CA 中部署 SQLite 数据库，用于处理 CA 的数据库必备操作，包括跟踪每个用户的注册数和任何撤销的证书。

**排序节点注意事项**
- 排序服务与 Hyperledger Fabric V1.2 的任何组件兼容。
- 此 Helm 图表部署 SOLO 排序服务的一个实例（一个排序节点）。请注意，无法在通道上部署多个 SOLO 排序节点以使排序服务具有高可用性。这是 SOLO 排序服务被视为适合用于开发环境而不适合用于生产环境的一个原因。但是，您可以将 SOLO 排序服务的多个实例部署到不同网络（即，使用不同的联盟）。

**同级注意事项**

- 只能将同级连接到 Fabric V1.1 或 V1.2.1 级别的区块链网络。可以通过打开“网络监视器”中的[“网络首选项”窗口](../v10_dashboard.html#network-preferences)来找到 Hyperledger Fabric 版本。请遵循有关从入门套餐或企业套餐网络中检索同级连接信息的[指示信息](#starter-enterprise-network-endpoints)进行操作。
- 同级的数据库类型必须匹配区块链网络的数据库类型，即 LevelDB 或 CouchDB。
- CouchDB Fauxton 接口在同级上不可用。
- 当前不支持将 [Gossip](../glossary.html#gossip) 用于同级。这意味着也不支持依赖于 Gossip 的 Fabric 功能，例如[专用数据 ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/private-data-arch.html "专用数据") 和[服务发现 ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/discovery-overview.html "服务发现")。

## 系统先决条件
{: #prerequisites}

{{site.data.keyword.blockchainfull_notm}} Platform for ICP 支持以下操作系统：
- Red Hat Enterprise Linux (RHEL) 7.3、7.4 和 7.5
- Ubuntu 18.04 LTS 和 16.04 LTS
- SUSE Linux Enterprise Server (SLES) 12 SP3

{{site.data.keyword.blockchainfull_notm}} Platform for ICP Helm 图表已经过验证，可以使用以下工作程序节点和备用存储器在 Ubuntu Linux 上的 {{site.data.keyword.cloud_notm}} Private V3.1.0 集群中运行：

- **LinuxONE 和 IBM Z**：z/VM 和 KVM（使用 NFS）。
- **x86**：Linux 64 位（使用 GlusterFS）。

## 许可和定价
{: #ibp-icp-license-pricing}
{{site.data.keyword.blockchainfull_notm}} Platform for {{site.data.keyword.cloud_notm}} Private 包含两个版本，一个是付费产品，可从 Passport Advantage 下载，一个是免费 Community Edition，在 [GitHub ![外部链接图标](images/external_link.svg "外部链接图标")](https://github.com/IBM/charts/tree/master/stable/ibm-blockchain-platform-dev "IBM/charts") 中提供。

### 许可
{: #ibp-icp-license}

{{site.data.keyword.blockchainfull_notm}} Platform on {{site.data.keyword.cloud_notm}} Private 交付的组件需要在您自己的基础架构上运行区块链网络，并作为 ICP 应用程序进行部署。您可以访问 PPA 并[下载 Helm 图表](/docs/services/blockchain/howto/helm_install_icp.html)。购买中包含 {{site.data.keyword.blockchainfull_notm}} 的技术支持。

{{site.data.keyword.blockchainfull_notm}} Platform on IBM Cloud Private Community Edition 是一个免费产品，适用于评估和试验，并作为 ICP 应用程序进行部署。请勿将 Community Edition 用于生产环境。{{site.data.keyword.blockchainfull_notm}} Platform 不提供对 Community Edition 的支持。您可以访问 [GitHub 包 ![外部链接图标](images/external_link.svg "外部链接图标")](https://github.com/IBM/charts/blob/master/repo/stable/ibm-blockchain-platform-dev-1.0.0.tgz "IBM/charts") 并下载 Helm 图表。

### 定价
{: #ibp-icp-pricing}

{{site.data.keyword.blockchainfull_notm}} Platform on ICP 的定价基于使用的虚拟处理器核心 (VPC) 的数量。VPC 可以是分配给虚拟服务器的虚拟核心，也可以是非分区服务器中的物理处理器核心。您必须为可用于 {{site.data.keyword.blockchainfull_notm}} Platform 的每个 VPC 获取许可权利。<!-- A VPC is a unit of measurement by which a program can be licensed.-->

有关如何确定 VPC 使用数的更多信息，请参阅 IBM Knowledge Center 中有关[“虚拟处理器核心 (VPC)”![外部链接图标](images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SS8JFY_9.2.0/com.ibm.lmt.doc/Inventory/overview/c_virtual_processor_core_licenses.html "虚拟处理器核心 (VPC)") 的文章。可以使用 [IBM License Metric Tool](https://www.ibm.com/support/knowledgecenter/en/SS8JFY_9.2.0/com.ibm.lmt.doc/welcome/LMT_welcome.html) 来配置和创建可用于确定需要许可的 VPC 数的报告。


## 安装 {{site.data.keyword.blockchainfull_notm}} Platform for ICP
{: #ibp-icp-install}

{{site.data.keyword.blockchainfull_notm}} Platform on {{site.data.keyword.cloud_notm}} Private 作为 Helm 图表文件交付，可安装在本地 ICP 集群中。安装 Helm 图表之后，您可在 ICP 目录中发现 {{site.data.keyword.blockchainfull_notm}} Platform 显示为应用程序。

- {{site.data.keyword.blockchainfull_notm}} Platform for ICP 通过 Passport Advantage 交付。您需要具有必需的许可证才能访问 [Passport Advantage Online](https://www.ibm.com/software/passportadvantage/pao_customer.html)。购买时，将包含对 {{site.data.keyword.blockchainfull_notm}} Platform 的技术支持。

- {{site.data.keyword.blockchainfull_notm}} Platform Community Edition 用于探索、开发和测试。IBM Blockchain Platform for ICP 的这一免费版本可通过 GitHub 访问。**注：**{{site.data.keyword.blockchainfull_notm}} Platform 不提供对 Community Edition 的支持。

有关如何安装 Helm 图表和必需的必备软件的指示信息，请参阅[安装 {{site.data.keyword.blockchainfull_notm}} Platform on {{site.data.keyword.cloud_notm}} Private](/docs/services/blockchain/howto/helm_install_icp.html)。

如果您是 {{site.data.keyword.cloud_notm}} Private 的新用户，并且希望了解有关安装和部署 ICP 的信息和技巧，请参阅[设置 {{site.data.keyword.cloud_notm}} Private](/docs/services/blockchain/ICP_setup.html)。

安装 {{site.data.keyword.blockchainfull_notm}} Platform for ICP 后，需要分别部署网络的每个组件。不能同时部署多个组件。请访问 [{{site.data.keyword.blockchainfull_notm}} Platform for ICP 部署指南](/docs/services/blockchain/ibp_for_icp_deployment_guide.html)，以在开始之前了解创建或加入区块链网络的最佳实践。然后，在以下各部分中查看用于部署和操作各个组件的步骤。

### 在防火墙后安装 {{site.data.keyword.blockchainfull_notm}} Platform for ICP
{: #ibp-icp-firewall}

您可以在防火墙后部署 {{site.data.keyword.blockchainfull_notm}} Platform 组件，而无需访问公用因特网。下载的 Helm 图表包中包含 {{site.data.keyword.blockchainfull_notm}} Platform 使用的所有 Fabric 组件 Docker 映像，无需在部署期间从 DockerHub 进行拉取。

{{site.data.keyword.blockchainfull_notm}} Platform Community Edition Helm 图表包不包含必需的 Fabric 组件 Docker 映像。它配置为在部署期间从 DockerHub 下载这些映像，因此在无法访问公用因特网时会失败。但是，您可以执行一些额外的步骤，在防火墙后部署 Community Edition 组件。有关更多信息，请参阅[在防火墙后安装 Community Edition](/docs/services/blockchain/howto/helm_install_icp.html#helm-install-prereqs-firewall)


## 关于 ICP 上的认证中心
{: #ibp-icp-ca}

认证中心 (CA) 在网络上提供身份。CA 可被视为类似于公证方，充当多个参与方之间的信任锚点。给予网络中的所有实体一个根 CA 签署的证书，该证书封装他们的数字身份。此证书是在网络上执行的所有签名和验证操作的信任根。有关认证中心及其在区块链网络中所扮演角色的更多信息，请参阅[区块链组件概述](/docs/services/blockchain/blockchain_component_overview.html)。

CA 将验证网络中需要部署的其他组件的身份，然后颁发证书。因此，您需要先为组织部署 CA，然后再部署其他组件。

- 要了解如何在安装 Helm 图表后配置和部署 CA，请参阅[在 {{site.data.keyword.cloud_notm}} Private 中部署认证中心](/docs/services/blockchain/howto/CA_deploy_icp.html)。

- 要了解如何使用 CA 来生成证书并完成部署其他组件的必备步骤，请参阅[在 {{site.data.keyword.cloud_notm}} Private 中操作认证中心](/docs/services/blockchain/howto/CA_operate.html)。

## 关于 ICP 上的排序节点
{: #ibp-icp-orderer}

排序节点可在区块链网络中对客户机进行认证、对事务排序和广播事务。排序节点是基于 Hyperledger Fabric 的区块链网络的公共绑定。因此，创建网络的组织需要部署并启动“排序服务”（执行排序的节点或节点集合），然后其他组织才能部署其同级，加入通道并开始在网络上进行事务处理。有关排序节点及其在区块链网络中所扮演角色的更多信息，请参阅[区块链组件概述](/docs/services/blockchain/blockchain_component_overview.html#orderer)。

如果您是区块链网络的创始人，那么需要部署排序节点。部署排序节点后，您可以邀请其他组织加入联盟，随后这些组织可以创建自己的通道。

- 要了解如何在安装 Helm 图表后配置和部署排序节点，请参阅[在 ICP 中部署排序节点](/docs/services/blockchain/howto/orderer_deploy_icp.html)。

- 要了解如何构建联盟，请参阅[在 ICP 上操作排序节点](/docs/services/blockchain/howto/orderer_operate.html)。

## 关于 ICP 上的同级
{: #ibp-icp-peer}

同级是网络的基本元素，因为它们用于托管分类帐和智能合同。智能合同和分类帐分别用于封装网络中的共享过程和共享信息。有关同级及其在区块链网络中所扮演角色的更多信息，请参阅[区块链组件概述](/docs/services/blockchain/blockchain_component_overview.html)。

- 准备好加入网络时，即可以部署同级，同级将加入通道，对事务背书以及存储通道分类帐。有关在 ICP 上部署同级以连接到 ICP 上的其他组件的信息，请参阅[在 ICP 上部署同级](/docs/services/blockchain/howto/peer_deploy_icp.html)。有关在 ICP 上部署同级以连接到入门套餐或企业套餐网络的信息，请参阅[部署将连接到入门套餐或企业套餐的同级](/docs/services/blockchain/howto/peer_deploy_ibp.html)。

- 设置了同级后，需要完成若干操作步骤，然后才能将事务提交到区块链网络的分布式分类帐，并读取该分布式分类帐。

  - 如果要将同级连接到在 ICP 上部署的 {{site.data.keyword.blockchainfull_notm}} Platform，请参阅[在使用多云网络的 {{site.data.keyword.cloud_notm}} Private 上操作同级](/docs/services/blockchain/howto/peer_operate_icp.html)。
  - 如果要将同级连接到在 {{site.data.keyword.cloud_notm}} 上部署的入门套餐或企业套餐网络，请参阅[在使用入门套餐或企业套餐的 {{site.data.keyword.cloud_notm}} Private 上操作同级](/docs/services/blockchain/howto/peer_operate_ibp.html)。


## 安全注意事项
{: #icp-security}

由于这些组件是在 {{site.data.keyword.cloud_notm}} 外部部署的，因此您负责管理其安全性。这包括重要安全性领域，例如密钥管理和数据加密。考虑组件的安全性时，请查看以下主题。

### 数据安全性
{: #icp-security-data}

{{site.data.keyword.blockchainfull_notm}} Platform 入门套餐和企业套餐使用的是基于[对称密钥加密 ![外部链接图标](images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSB23S_1.1.0.14/gtps7/s7symm.html "对称加密") 的全盘加密来保护网络使用的所有数据。您必须在自己的环境中执行类似的步骤来保护同级数据。

状态数据库（无论使用的是 LevelDB 还是 CouchDB）中的数据未加密。可以使用应用程序级别加密来保护状态数据库中的静态数据。

### 数据存储位置
{: #data-residency}

数据存储位置需求可能要求所有区块链分类帐数据的处理和存储始终限制在一个国家或地区境内（或其他某个定义的边界内）。有关如何实现数据存储位置的更多信息，请参阅[数据存储位置](#icp-data-residency)。

### 密钥管理
{: #icp-security-key-management}

密钥管理是安全性中至关重要的一个方面。如果专用密钥泄漏或丢失，那么恶意参与者可能有能力访问您的数据和功能。IBM 使用称为[硬件安全模块](/docs/services/blockchain/glossary.html#hsm) (HSM) 的物理设备来存储 {{site.data.keyword.blockchainfull_notm}} Platform 企业套餐网络的专用密钥。

在 ICP 上部署组件时，您负责管理专用密钥。虽然 {{site.data.keyword.blockchainfull_notm}} Platform 会生成专用密钥，但这些密钥不会存储在 Platform 上。因此，必须安全地存储密钥，避免密钥泄露。您可以在同级 MSP 的 keystore 文件夹（位于组件内的 `/mnt/crypto/peer/peer/msp/keystore/` 目录）中找到组件的专用密钥。有关同级内的证书的更多信息，请参阅[在 {{site.data.keyword.blockchainfull_notm}} Platform 上管理证书](../certificates.html)教程中的[成员资格服务提供者](/docs/services/blockchain/certificates.html#msp)部分。

您可以使用“密钥代管”来恢复丢失的专用密钥。这需要在丢失任何密钥之前执行。如果无法恢复专用密钥，那么需要通过向认证中心注册新的身份来获取新的专用密钥。您还应该从加入的任何通道中除去并替换 signCert。

### TLS
{: #icp-security-tls}

[传输层安全性 ![外部链接图标](images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_7.1.0/com.ibm.mq.doc/sy10660_.htm "SSL 或 TLS 握手概述") (TLS) 嵌入在 Hyperledger Fabric 的信任模型中。{{site.data.keyword.blockchainfull_notm}} Platform 上的所有入门套餐和企业套餐组件都使用 TLS 进行认证并相互通信。因此，Platform 上的网络组件需要能够完成与同级的 TLS 握手。这样做的其中一个影响是，您需要在从客户机应用程序到同级之间的防火墙中启用传递（例如，使用白名单来实现）。

<!--
You can use Mutual TLS, which requires two way (server-client) rather than one way (server only) authentication, to secure the communication between your application and Enterprise Plan networks. You can use the Network Monitor [to enable mutual TLS](../v10_dashboard.html#mutual-tls) for peers on {{site.data.keyword.blockchainfull_notm}} Platform. To enable Mutual TLS on your peer, follow the instructions to [enable mutual-TLS for peer nodes ![External link icon](images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.1/enable_tls.html "Securing Communication with Transport Layer Security") in the Hyperledger Fabric documentation. It is strongly recommended that you enable mutual-TLS for your applications.
-->

### 成员资格服务提供者配置
{: #icp-security-MSP}

{{site.data.keyword.blockchainfull_notm}} Platform 的组件通过成员资格服务提供者 (MSP) 使用身份。MSP 将 CA 颁发的证书与网络和通道角色相关联。有关 MSP 如何用于同级的更多信息，请参阅[成员资格服务提供者 (MSP)](/docs/services/blockchain/certificates.html#msp)。

### 应用程序安全性
{: #icp-security-appl}

由于所有链代码调用都已签名，因此 Fabric 会管理应用程序安全性。此外，Fabric 还包含基于 ACL 的应用程序级别检查。

## 数据存储位置
{: #icp-data-residency}

由于区块链网络并不关注所处理的数据类型，因此有时必须采取额外的步骤来确保某些类型数据的安全。数据存储位置的最常见需求与特定国家或地区的法律相关，这要求在 IT 系统中处理和存储的所有数据必须保留在特定国家或地区境内。与此类似，受到高度监管的行业（例如，政府、医疗保健和金融服务）中的某些公司要求必须将数据完全存储在其防火墙之后。因此，为实现数据存储位置，区块链网络的所有组件都必须属于相同[通道](/docs/services/blockchain/glossary.html#channel)并且位于单个国家或地区境内。

为满足数据存储位置需求，重要的是了解作为 {{site.data.keyword.blockchainfull_notm}} Platform 基础的 Hyperledger Fabric 体系结构。该体系结构主要围绕着三个关键组件：排序服务（由排序节点组成）、认证中心 (CA) 和同级。同级以区块形式接收来自排序服务的已排序状态更新并维护状态和分类帐。因此，同级和排序服务具有直接关系。分类帐包含所有事务日志所包含的所有密钥和数据的最新值。

此外，客户机应用程序使用 [Fabric SDK](v10_application.html#using-the-fabric-sdks) 以将事务发送给同级和排序服务。这些事务包含[读/写集语义 ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/readwrite.html "读/写集语义") 数据，其中包含有关分类帐的密钥/值对。

如果要求国家或地区内的数据存储位置，那么排序服务、同级和客户机应用程序必须位于同一国家或地区内。在 {{site.data.keyword.cloud_notm}} 中创建 {{site.data.keyword.blockchainfull_notm}} Platform 网络时，您可以选择网络的位置。<!--For a Starter Plan network, you can select from US South, United Kingdom, and Sydney. For an Enterprise Plan network, you can select from currently available locations, which include Dallas, Frankfurt, London, Sao Paulo, Tokyo, and Toronto. -->有关区域和位置的更多信息，请参阅 [{{site.data.keyword.blockchainfull_notm}} Platform 区域和位置](/docs/services/blockchain/reference/ibp_regions.html)。

### 数据存储位置的用例

假设 {{site.data.keyword.blockchainfull_notm}} Platform 网络包含排序节点和 CA，并且有包含四个组织的联盟。这些组织有一个或多个同级节点，所有四个组织都属于一个通道，网络的所有组件都位于部署了 {{site.data.keyword.blockchainfull_notm}} Platform 网络的区域（例如，法兰克福）中。最后，与同级进行交互的客户机应用程序也位于德国境内。

在此示例中，保持了数据存储位置。

![所有组件位于同一国家或地区时的数据存储位置](images/remote_peer_data_res_1.png "所有组件位于同一国家或地区时的数据存储位置")
*图 1. 所有组件位于同一国家或地区时的数据存储位置*

现在，让我们来考虑**分布式同级**加入其中一个组织时会产生的影响。分布式同级可以位于网络的其余部分所在的相同区域中，也可以位于 {{site.data.keyword.blockchainfull_notm}} Platform 网络中该区域以外的任何位置：

-	如果同级位于网络的其余部分所在的相同国家或地区，那么保持了数据存储位置。所有分类帐数据都会保留在德国境内，如上面**图 1** 所示。
-	如果同级位于其他国家或地区（例如，美国），那么数据存储位置不再得到保持，因为同级分类帐上的数据在该国家或地区境外共享。

要解决此问题，可以使用**通道**将数据隔离到网络上的一部分同级。{{site.data.keyword.blockchainfull_notm}} Platform 网络包含跨越国家或地区边境的分布式同级和排序节点时，通道可用于将分类帐数据与具有该国家或地区境外的同级的组织相隔离。

**注：**排序节点总是位于您选择用于托管网络的数据中心区域内。不能有多个排序节点跨越国家或地区边境。但是，同级可位于数据中心内或位于 {{site.data.keyword.cloud_notm}} 之外的远程位置。

![同级位于 IBM Blockchain Platform 区域所在国家或地区之外时的数据存储位置](images/remote_peer_data_res_2.png "同级位于 IBM Blockchain Platform 区域所在国家或地区之外时的数据存储位置")
*图 2. 同级位于 IBM Blockchain Platform 区域所在国家或地区之外时的数据存储位置*

在**图 2** 中，`OrgC` 和 `OrgD` 不需要数据存储位置。事实上，`OrgD` 现在包含两个位于*美国*的分布式同级 `OrgD-peer1` 和 `OrgD-peer2`。因此，为了使 `OrgA`、`OrgB` 及其各自的客户机应用程序和位于德国的同级能够隔离通道 `X` 上的分类帐数据，针对 `OrgC` 和 `OrgD` 创建了新的通道 `Y`。

要更深刻地了解 {{site.data.keyword.blockchainfull_notm}} Platform 网络上的数据流，请参阅[有关事务流的 Fabric 文档 ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/txflow.html "事务流")。

将来，Hyperledger Fabric 中的新技术将使用[专用数据集合 ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/private-data/private-data.html "专用数据集合") 和“零知识证明”来提高进一步实现数据存储位置的能力。

- 专用数据集合确保专用数据在同级之间共享（通过 Gossip 协议）时，只有授权的同级（例如，国家或地区境内的同级）才能看到该数据。数据存储在同级的专用数据库中。此处不涉及排序服务，并且排序服务也不会看到专用数据。会将散列化的此数据写入到通道上每个同级的分类帐。用于状态验证的散列充当事务证据，并且可用于审计目的。专用数据可用于 {{site.data.keyword.blockchainfull_notm}} Platform 上在 Fabric V1.2.1 上运行的网络。但是，由于 ICP 网络当前不使用 Gossip，因此专用数据功能不可用于在 ICP 中运行的同级。

- 通过“零知识证明”(ZKP)，“证明者”能使“验证者”相信自己知道这些保密信息，而又不向验证者泄露保密信息本身。

您可以在有关 [Private and confidential transactions with Hyperledger Fabric ![外部链接图标](images/external_link.svg "外部链接图标")](https://www.ibm.com/developerworks/cloud/library/cl-blockchain-private-confidential-transactions-hyperledger-fabric-zero-knowledge-proof/index.html "Private and confidential transactions with Hyperledger Fabric") 的白皮书中了解这些技术的更多相关信息。

## 获取支持
{: #ibp-icp-support}

有关数字支持产品的信息，请参阅 IBM Blockchain Platform 支持[资源和支持论坛](/docs/services/blockchain/ibmblockchain_support.html#resources)。

### {{site.data.keyword.blockchainfull_notm}} Platform for ICP

如果您已购买 IBM Blockchain Platform for ICP 许可证并且要与客户支持人员联系，请参阅有关[访问 IBM 支持社区并开具支持凭单 ![外部链接图标](images/external_link.svg "外部链接图标")](https://www-01.ibm.com/support/docview.wss?uid=ibm10740041 "{{site.data.keyword.blockchainfull_notm}} Platform for ICP 支持") 的信息。

### {{site.data.keyword.blockchainfull_notm}} Platform Community Edition

Community Edition 用于探索、开发和测试。IBM Blockchain Platform 不提供对 {{site.data.keyword.blockchainfull_notm}} Platform Community Edition 的支持。

如果遇到与区块链组件相关的问题，您可以使用免费区块链开发人员资源和支持论坛，并从 IBM 和 Fabric 社区获取帮助。有关更多信息，请参阅[区块链资源和支持论坛](/docs/services/blockchain/ibmblockchain_support.html#resources)。

有关与 IBM Cloud Private 相关的问题，您可以利用 [ICP 提供的免费数字支持和付费支持](https://www.ibm.com/developerworks/community/blogs/fe25b4ef-ea6a-4d86-a629-6f87ccf4649e/entry/Learn_more_about_IBM_Cloud_Private_Support?lang=en_us)。
