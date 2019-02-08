---

copyright:
  years: 2017, 2018
lastupdated: "2018-12-07"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}

# 新增功能
{: #whatsnew}

***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

## 2018 年 12 月 7 日

*操作入门套餐网络*和*操作企业套餐网络*主题组合并替换为关于[使用网络监视器](/docs/services/blockchain/v10_dashboard.html)的单个教程。

## 2018 年 11 月 27 日

{{site.data.keyword.blockchainfull}} Platform 发布了 {{site.data.keyword.blockchainfull_notm}} Platform for {{site.data.keyword.cloud_notm}} Private (ICP)。ICP 是一个应用程序平台，用于开发和管理基于 Kubernetes 的容器化应用程序，它允许用户在 x86、LinuxONE 和 IBM Z 上部署认证中心 (CA)、排序节点和同级。{{site.data.keyword.blockchainfull_notm}} Platform for ICP 基于 Hyperledger Fabric V1.2.1，并使用 Kubernetes Helm 图表进行部署。安装 Helm 图表后，您可在 ICP 目录中发现它显示为捆绑服务。请注意，[此产品](/docs/services/blockchain/ibp-for-icp-about.html)更适合经验丰富的 Fabric 用户。

有关 ICP 的更多信息，请参阅 [{{site.data.keyword.cloud_notm}} Private V3.1.0 文档 ![外部链接图标](images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/kc_welcome_containers.html "{{site.data.keyword.cloud_notm}} Private V3.1.0 文档 ")。

此外，{{site.data.keyword.blockchainfull_notm}} Platform for ICP 发布之后，IBM Blockchain Platform Remote Peer (Beta) 程序即宣告终结。您仍可以在 ICP 或 AWS 中部署同级，并将其连接到入门套餐或企业套餐网络。有关更多信息，请参阅[针对入门套餐或企业套餐部署同级](/docs/services/blockchain/howto/peer_deploy_ibp.html)以及[在 AWS 中部署同级](/docs/services/blockchain/howto/remote_peer_aws.html)。这些同级被视为**分布式**同级而不是远程同级，因为部署由客户进行管理，因此没有一个中央位置可作为远程位置的基础。

此发布还首次对文档目录进行了一些改进。如果您是入门套餐或企业套餐用户，那么您的内容仍可在**了解**、**方法**、**参考**和**帮助**部分中找到，并且链接仍然相同。内容可能位于目录的不同子节中。

## 2018 年 11 月 13 日

{{site.data.keyword.blockchainfull_notm}} Platform 发布了 {{site.data.keyword.blockchainfull_notm}} Platform for Amazon Web Services (AWS)。

{{site.data.keyword.blockchainfull_notm}} Platform for AWS 允许您在 AWS Cloud 中运行同级，并将同级连接到 {{site.data.keyword.cloud_notm}} 中的现有区块链网络。AWS 中的同级可以利用现有网络中的
连接概要文件和其他区块链组件，同时使您可以更灵活地将同级与 {{site.data.keyword.cloud_notm}} 之外的其他应用程序并置在一起。有关更多信息，请参阅[关于 {{site.data.keyword.blockchainfull_notm}} Platform for Amazon Web Services](/docs/services/blockchain/howto/remote_peer.html)。

## 2018 年 10 月 4 日

{{site.data.keyword.blockchainfull_notm}} Platform 将入门套餐从 Hyperledger Fabric V1.1.0 升级到 V1.2.1。有关更多信息，请参阅[关于入门套餐](/docs/services/blockchain/starter_plan.html)。

**重要信息：**Fabric V1.2.1 当前与使用 V1.1.0 同级的 Remote Peer beta 版不兼容。入门套餐网络（在 2018 年 10 月 4 日之前创建，并基于 Fabric V1.1.0）不受影响，仍然可以与 Remote Peer beta 版配合使用。{{site.data.keyword.blockchainfull_notm}} Platform 将更新 Remote Peer beta 版以使用 V1.2.1 同级，该同级将与新的入门套餐网络（Fabric V1.2.1 级别）和企业套餐网络（Fabric V1.1.0 级别）兼容。在 Remote Peer beta 版更新到 Fabric V1.2.1 之前，不要尝试使用新的 V1.2.1 入门套餐网络部署 V1.1.0 远程同级。

## 2018 年 9 月 4 日

{{site.data.keyword.blockchainfull_notm}} Platform 发布了 Remote Peer 产品 beta 版。Remote Peer 产品基于 Hyperledger Fabric V1.1.0。通过使用 Remote Peer，您可以在自己的 IBM Cloud Private (ICP) 或 Amazon Web Services (AWS) Cloud 环境中运行 {{site.data.keyword.blockchainfull_notm}} Platform 同级节点。有关更多信息，请参阅[关于远程同级](/docs/services/blockchain/howto/remote_peer.html)。

## 2018 年 6 月 15 日

{{site.data.keyword.blockchainfull_notm}} Platform 发布了入门套餐 GA。有关更多信息，请参阅[关于入门套餐](/docs/services/blockchain/starter_plan.html)。

## 2018 年 5 月 15 日

{{site.data.keyword.blockchainfull_notm}} Platform 将企业套餐从 Hyperledger Fabric V1.0.0 升级到 V1.1.0。有关更多信息，请参阅[关于企业套餐](/docs/services/blockchain/enterprise_plan.html)。

## 2018 年 3 月 18 日

{{site.data.keyword.blockchainfull_notm}} Platform 发布了企业套餐 beta 版。企业套餐为您提供了开发和测试环境，以在 {{site.data.keyword.blockchainfull_notm}} Platform 网络中进行学习和实践。有关更多信息，请参阅[关于入门套餐](/docs/services/blockchain/starter_plan.html)。

## 2017 年 8 月 11 日

{{site.data.keyword.blockchainfull_notm}} Platform 替换了 {{site.data.keyword.blockchainfull_notm}} on Cloud，并发布了企业套餐。有关更多信息，请参阅[关于企业套餐](/docs/services/blockchain/enterprise_plan.html)。
