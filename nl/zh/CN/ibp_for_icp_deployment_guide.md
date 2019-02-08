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

# {{site.data.keyword.blockchainfull_notm}} Platform for ICP 部署指南

***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

基于 [Hyperledger Fabric](https://hyperledger-fabric.readthedocs.io/en/release-1.2/) 的区块链网络可以采用几乎无限制的一系列配置进行部署，以支持大量用例。尽管存在此灵活性，还是存在若干最佳实践，特别是在设置和部署网络组件的正确**顺序**方面。

本部署指南显示了在 {{site.data.keyword.cloud_notm}} Private (ICP) 上设置正常运行的 {{site.data.keyword.blockchainfull}} Platform 网络的正确顺序，以及在部署时的最佳实践和考虑事项。但是，如果您计划仅设置一个正常运行的 CA、排序节点或同级，基本规则仍然适用。请注意，SOLO 排序服务仅用于部署单个排序节点，不适用于生产环境。运行 SOLO 的任何网络或通道都不能视为“生产”环境。但是，可以在生产环境中部署同级和 CA，尤其是当它们高度可用时。

**重要信息**：部署 IBP on ICP 的过程非常复杂，并且需要您掌握高水平的 Fabric 专业知识。如果您不熟悉 Fabric、IBP 或 ICP，并且目标是设置开发环境或概念验证，请考虑转而查看[入门套餐](/docs/services/blockchain/starter_plan.html)。另请注意，并非所有可能的部署配置都受支持。鉴于牵涉的复杂程度，最好让 Fabric 专家完成部署过程并将组件连接在一起，然后再将管理工作移交其他方。这些专家是本指南以及我们通常针对 Helm 图表组件提供的文档的目标受众。

## 确定网络配置

区块链网络的结构必须由用例指定。这些基本业务决策在不同的情况下会有所不同，不过我们还是来考虑几种情况。

* 设置**开发环境**

  这正是[入门套餐](/docs/services/blockchain/starter_plan.html)构建用于的环境，但是如果您想要在开发环境中管理自己的组件，您将需要一个基本配置，其中包含一个排序节点和一个同级（每个组织有一个 CA）。对于每个组织，您可能决定只拥有一个排序节点和一个同级，类似于 [fabcar 网络](https://hyperledger-fabric.readthedocs.io/en/release-1.2/understand_fabcar_network.html)的配置。同样，您可能决定对于所有这些组件，只需要一个管理员用户，而不是为每个组件创建不同的用户。

* 为新网络或现有网络部署**生产组件**

  生产组件和生产网络的需求不同于开发环境或概念验证。一方面，高可用性需要优先考虑。SOLO 排序服务只包括单个排序节点（因而会成为单点故障），根据定义，它不适用于生产环境。

**注：**此部署指南不会遍历每个迭代和可能的网络配置，但会提供需要考虑的通用准则和规则。

## 设置 ICP Kubernetes 集群

确定网络结构后，在 ICP 上设置 Kubernetes 集群以适应用例。有关更多信息，请参阅[设置 {{site.data.keyword.cloud_notm}} Private](/docs/services/blockchain/ICP_setup.html)。

## 设置 CA

在基于 Fabric 的区块链网络中，首先必须部署的组件是 CA。这是因为组件的配置必须至少包含一个授权操作该组件的用户身份，然后才能部署该组件。

这在概念上有时可能难以理解。例如，笔记本电脑之类的新电子设备通常可供任何人使用。第一个开启该设备的人员假定是“所有者”，因此该人员可以注册并创建每个后续用户必须知道的密码。但是，基于 Fabric 的区块链网络中的组件不是这样设置的。同级和排序节点在第一次部署时必须内置有管理信息。

**建议对每个组织部署一个 CA。**例如，如果部署与一个组织相关联的三个同级，以及与另一个组织相关联的一个排序节点，那么需要部署两个 CA。一个 CA 用于同级组织，另一个 CA 用于排序节点组织。

为了防止 CA 的无限回归（其中每个 CA 必须链接到另一个 CA，以此类推），CA 关联有缺省用户名和密码对。您在部署 CA 时指定此用户名和密码。此 CA 是“根 CA”，是组件的信任根。在生产网络中，建议您部署至少一个从此根 CA 获取其证书的其他 CA，并使用该 CA（称为“中间 CA”）来向用户和组件颁发证书。但是，在此发行版中，不支持使用中间 CA。

每个组织都必须拥有用于注册的 CA 和 TLS CA。在 ICP 上部署 CA 时，缺省情况下还会在同一容器中部署 TLS CA。此 TLS CA 生成并管理 TLS 证书。入门套餐或企业套餐网络上的 CA 不包含 TLS CA，但它包含用于注册的 CA。
因此，如果计划将同级连接到入门套餐或企业套餐网络，那么在部署同级之前，还必须[在 ICP 中部署 CA](/docs/services/blockchain/howto/CA_deploy_icp.html)，该 CA 将充当该同级的 TLS CA。另请参阅[将 ICP 同级连接到 IBP 的先决条件](/docs/services/blockchain/howto/peer_deploy_ibp.html#prerequisites-peer-ibp)。请注意，TLS CA 仅用于颁发证书，并且可以在该活动完成时关闭。

有关 TLS 的更多信息，请参阅 Hyperledger Fabric 文档中的[使用传输层安全性 (TLS) 保护通信安全 ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.3/enable_tls.html "使用传输层安全性 (TLS) 保护通信安全")。

### 为排序节点和同级准备 MSP

由于 IBP for ICP 过程非常复杂，因此建议在初始设置期间使用单个管理员用户身份作为所有网络组件节点的管理员。通过确保一个用户可以设置各种组件之间的配置和连接来确保它们正确运行，这应该会减少部署和连接错误。但是，每个组件具有不同的证书极其重要。有时很容易忽视这一差异性。对事务建议签名的实体不是同级的管理员，而是**同级自身**。因此，同级必须注册，并拥有连接到任何内容的证书以及可用于生成特定类型签名的专用密钥。有关基于 Fabric 的区块链网络中的身份和许可权的更多信息，请参阅 Fabric 文档中的[身份 ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.3/identity/identity.html "身份") 和[成员资格 ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.3/membership/membership.html "成员资格")。

Fabric CA 客户机（应遵循[操作 CA 的文档](/docs/services/blockchain/howto/CA_operate.html#fabric-ca-client)中的指示信息来安装）用于注册和登记身份。此身份是您将部署的网络组件的管理员，允许您更改配置以及将组件连接在一起。如果以后要将这些组件的管理转移给其他用户，可为这些组件注册并登记新的管理员，并在需要时除去您自己的管理员身份。

当部署排序节点或同级时，与该排序节点或同级相关联的 `init` 容器将使用 Kubernetes 私钥对象来为组件创建 MSP。要了解如何创建私钥对象，请参阅[操作 CA](/docs/services/blockchain/howto/CA_operate.html)。如上所述，请记住，您需要为每个组织设置一个 CA 并重复此流程。

## 部署排序节点和同级

创建 Kubernetes 私钥后，您已经准备就绪，可部署组件。如果您的计划是建立通道，建议您在部署任何同级之前部署排序节点。确保对所有组件使用不同的组织名称。

- **[部署排序节点](/docs/services/blockchain/howto/orderer_deploy_icp.html)**。请注意，目前仅支持 SOLO 排序服务。在部署过程中，您可以选择与计算相关的选项。

- **[部署同级以将其连接到 ICP 排序节点](/docs/services/blockchain/howto/peer_deploy_icp.html)**。用于部署同级的若干选项（包括数据库类型）可以在 Helm 图表中指定。请确保同级的组织 MSP 标识不同于排序节点的组织 MSP 标识。

- **[部署同级以将其连接到 IBP 网络](/docs/services/blockchain/howto/peer_deploy_ibp.html)**。部署同级并将其连接到 {{site.data.keyword.cloud_notm}} 中[入门套餐](starter_plan.html)或[企业套餐](enterprise_plan.html)网络的过程是不同的，因为它使用“连接概要文件”和“网络监视器 UI”。请注意，同级所属的组织必须已加入该网络中的通道。另外，请确保同级的组织 MSP 标识不同于排序节点的组织 MSP 标识。

## 壮大网络

如果您的目标是设置开发环境或概念验证，那么您需要将同级组织添加到在排序节点部署期间创建的排序节点系统通道。这是一个利用每个组件的多步骤流程，记录在相关的操作主题中。

1. 您需要先创建组织并设置其 MSP 标识，然后才能添加该组织。请参阅[操作 CA](/docs/services/blockchain/howto/CA_operate.html#deploy-orderer-peer)。

2. 创建组织后，可以将其添加到排序节点系统通道。有关更多信息，请参阅[操作排序节点](/docs/services/blockchain/howto/orderer_operate.html#add-organizations-to-consortium)。

3. 组织列示在排序节点系统通道上时，该组织就是“联盟”的成员，能够创建应用程序通道，即执行事务所在的通道类型。有关如何创建通道的更多信息，请参阅[操作同级](/docs/services/blockchain/howto/peer_operate_icp.html#peer-icp-channeltx)。
