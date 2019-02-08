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

# 关于企业套餐


***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


{{site.data.keyword.blockchainfull}} Platform 企业套餐是一种生产就绪型产品，适用于希望创建或加入区块链网络以开展真实业务的组织。此套餐提供关键基础架构以及工具和支持，可轻松地开始使用高度安全的生产就绪型网络。2018 年 5 月 15 日，企业套餐从 Hyperledger Fabric V1.0 升级到 V1.1。2018 年 5 月 15 日之后创建的所有网络都为 Fabric V1.1 级别。但是，先前创建的网络将保持为 Fabric V1.0 级别。
{:shortdesc}

**注：**
- {{site.data.keyword.blockchainfull_notm}} Platform 企业套餐提供的是生产环境。如果您需要开发和测试环境，请参阅[关于入门套餐](/docs/services/blockchain/starter_plan.html)。
- {{site.data.keyword.blockchainfull_notm}} Platform 是 {{site.data.keyword.cloud_notm}} 上的平台服务，所有成员资格产品均遵循服务级别协议 (SLA) 上的 [{{site.data.keyword.cloud_notm}} 服务条款 ![外部链接图标](images/external_link.svg "外部链接图标")](http://www-03.ibm.com/software/sla/sladb.nsf/sla/bm "{{site.data.keyword.cloud_notm}} 服务条款")。企业套餐网络是跨**多个环境**进行供应的，这些环境分布在不同地理位置的多个数据中心。

对于即将启动网络的成员，IBM 提供图形用户界面来指导网络启动器完成设置和供应网络的关键步骤。这包括邀请其他成员和设置管控规则。有关更多信息，请参阅[管理企业套餐网络](/docs/services/blockchain/get_start.html)。一旦部署了网络，就可以使用交互式图形用户界面（即“网络监视器”）来监视网络的运行状况和活动，管理关键网络活动（包括新部署、添加/除去成员、链代码生命周期和通道管理），以及寻求技术支持。 有关更多信息，请参阅[使用网络监视器](v10_dashboard.html)。

立即注册以获取 [{{site.data.keyword.blockchainfull_notm}} 成员资格 ![外部链接图标](images/external_link.svg "外部链接图标")](https://console.bluemix.net/catalog/services/blockchain?env_id=ibm:yp:us-south&taxonomyNavigation=apps)。

{{site.data.keyword.blockchainfull_notm}} Platform 以关键的 Hyperledger Fabric 组件为基础构建，其中包括一个认证中心 (CA) 和至少 1 个同级（最多 6 个）。企业套餐还为网络成员提供崩溃容错 (CFT) Kafka 排序服务。

Fabric CA 是企业套餐随附的认证中心。每个成员提供了两个中间 CA，用于将成员资格授予网络。通过使用 CA，成员还可以为网络的用户提供成员资格（证书）。

请务必了解，要将事务处理附加到分类帐，涉及以下三个阶段：  
1. 事务处理模拟和支持（同级）
2. 订购（订购服务）
3. 验证和提交（同级）

成员所拥有的 Fabric 同级是应用程序的接口或网关，用于执行链代码，从而提供用于对分类帐执行事务处理的业务逻辑。必须支持所有事务处理。网络的其他成员会执行此支持。支持后，事务处理将发送到 IBM 提供的订购服务 (Kafka)。

除了核心区块链组件外，“企业成员资格”选项还提供了具有安全数据存储和通信 (TLS) 以及高可用性的基础架构。虽然 Fabric 网络共享这些基础架构资源，但是为网络中的 Fabric 组件节点提供了隔离，并且每个节点都在用于保护执行环境的安全 Docker 容器中执行。

必须确定的唯一方面是网络所需的同级的大小。此决策基于所需的通道数，以及每个通道的工作负载、内存使用情况和磁盘空间（存储器）。

您应该将企业套餐用于生产级别或接近生产级别的更稳定部署。对于测试用途，请使用[入门套餐](starter_plan.html)、[在 IBM Container Service 中进行开发](https://ibm-blockchain.github.io/)或[在本地安装 Docker 映像](http://hyperledger-fabric.readthedocs.io/en/release-1.1/build_network.html)。

<!--- The Enterprise plan provides the ordering service and CA. The membership fee is $1,000, and a per peer fee of $1,000 that is associated with the network. If you want to have high availability (HA), you must purchase an additional peer to provide the HA capabilities. For example, one organization (associated membership fee of $1,000) of two peers ($1,000 X 2 peers) with HA ($1,000 X 2 HA peers) requires a monthly charge of $5,000.  --->

## 定价  
要使用企业套餐，网络成员必须每月支付 1000 美元的成员资格费用，并且每月为网络中的每个同级额外支付 1000 美元。每月费用按日分派计费。例如，如果一个成员（关联的成员资格费用为 1,000 美元）有两个同级（每个同级的费用 1,000 美元 X 2 个同级），该成员每月需要支付 3,000 美元。如果该月有 30 天，那么该成员每天支付 100 美元（3,000 美元/30）。请注意，如果需要高可用性 (HA)，必须使必需的同级数量翻倍才能提供 HA 功能。

网络成员可以使用自己的 {{site.data.keyword.cloud_notm}} 帐户（其中包含用于创建网络实例的空间）来支付帐单。或者，一个网络成员可以支付网络中所有成员的帐单。有关如何为区块链网络付费的更多信息，请参阅[为网络付费](/docs/services/blockchain/howto/paying_mode.html)。
