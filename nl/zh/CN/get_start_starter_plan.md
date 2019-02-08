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
{:tip: .tip}

# 入门套餐使用入门
{: #getting-started-with-starter-plan}


***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


{{site.data.keyword.blockchainfull}} Platform 入门套餐提供了一个预先配置的区块链网络，只需单击一下就可使用。入门套餐供应一个许可的网络，缺省情况下配置有两个[组织](/docs/services/blockchain/glossary.html#organization)、每个组织一个[同级](/docs/services/blockchain/glossary.html#peer)，还有一个[通道](glossary.html#channel)。创建网络后，可以向网络扩展和添加更多组织和同级。
这些网络适合新接触 {{site.data.keyword.blockchainfull_notm}} Platform 的新手用户。<!--Note that it might cause extra cost if you exceed the default resource limits of two organizations and two peers.-->
{:shortdesc}

入门套餐支持您学习和培养使用 {{site.data.keyword.blockchainfull_notm}} Platform 的技能，运行样本应用程序，测试您自己的应用程序以及模拟多组织场景。本入门教程将指导您如何使用入门套餐在区块链网络上开始开发和事务处理。

如果您是 {{site.data.keyword.blockchainfull_notm}} Platform 和区块链新手，那么您可以通过查看基于开放式源代码 [Hyperledger Fabric](/docs/services/blockchain/reference/v10_fabric.html) 构建的网络的
[基本组件概述](blockchain_component_overview.html)，了解有关区块链的更多信息。您还可以查看 [Hyperledger Fabric 文档 ![外部链接图标](images/external_link.svg "外部链接图标")](http://hyperledger-fabric.readthedocs.io/en/release-1.2/blockchain.html "Introduction to Blockchain")。

**注**：{{site.data.keyword.blockchainfull_notm}} Platform 入门套餐是一个开发与测试环境，不适合生产工作负载。如果需要生产环境，请参阅[关于企业套餐](/docs/services/blockchain/enterprise_plan.html)。

## 概述

**构建您的联盟**

使用区块链进行构建的第一步是形成希望使用区块链进行事务处理的组织联盟，并邀请这些组织加入您的网络。试验该技术的入门套餐用户可以通过自行创建新的组织来模拟多组织联盟。

- [创建入门套餐网络](#creating-a-network)
- [邀请组织加入您的网络](#inviting-members)
- [加入入门套餐网络](#joining-a-network)

在加入您网络的组织的联盟内，您可以通过创建通道，私下与这些组织集进行事务处理。

- [创建通道](#create-channels)

**开发网络和应用程序**

形成组织联盟后，您需要编写链代码（又称为智能合同），其中包含网络的业务逻辑，并允许您与区块链分类帐上的数据进行交互。然后，您需要将 Fabric SDK 与这些智能合同一起使用，以便从客户端应用程序向网络提交事务。

- [开发链代码](#develop-chaincode)
- [使用 Fabric SDK 开发应用程序](#develop-apps)

您还可以使用 Hyperledger Composer（Hyperledger Fabric 的开放式源代码工具集）来开发业务逻辑，在网络上部署逻辑，并从应用程序连接到该逻辑。

- [使用 Hyperledger Composer 开发和部署定制业务网络](#develop-composer)

{{site.data.keyword.blockchainfull_notm}} Platform 提供使用 Hyperledger Composer 构建的样本应用程序，以及使用链代码构建的样本，您只需单击几次就可以将它们部署到您的入门套餐网络。

- [部署样本应用程序](#sample-applications)

**操作和管理网络**

{{site.data.keyword.blockchainfull_notm}} Platform 提供您可用于管理区块链网络的成员资格、生命周期和运行状况的工具和 API。这些功能中许多都可以通过“网络监视器”用户界面来进行访问。

- [监视网络资源](#monitoring-resources)
- [检索网络凭证和连接概要文件](#retrieving-network-credentials)
- [使用 Swagger API 管理网络](#swagger)
- [重置网络](#resetting-network)
- [从入门套餐迁移到企业套餐](#migrate)
- [删除或离开网络](#delete-network)


## 创建网络
{: #creating-a-network}

创建 {{site.data.keyword.blockchainfull_notm}} Platform 服务实例后，可以立即获取使用缺省配置的入门套餐[网络](/docs/services/blockchain/glossary.html#network)。

1. 在 {{site.data.keyword.cloud_notm}}“目录”中，找到 [Blockchain 服务 ![外部链接图标](images/external_link.svg "外部链接图标")](https://console.bluemix.net/catalog/services/blockchain)。**注**：您需要使用 {{site.data.keyword.cloud_notm}} 付费帐户登录。如果您没有帐户，请单击**注册以创建**按钮。在创建免费试用帐户后，通过转至 {{site.data.keyword.cloud_notm}} 控制台中的**管理** > **计费和使用情况** > **计费**，然后单击**添加信用卡**，将其升级为**现买现付**类型。
2. 选择 {{site.data.keyword.cloud_notm}} 中的区域以创建网络。
3. 选择 Cloud Foundry 组织和空间以创建网络。
4. 从价格套餐表中选择**入门成员资格套餐**。
5. 单击**创建**按钮。请注意，如果您受邀加入网络，那么会看到一个弹出式欢迎面板。要创建网络，请选择**继续至网络**，然后单击**继续**。要加入网络，请参阅[加入网络](#joining-a-network)中的步骤 5。
  现在您已准备好使用具有缺省配置的入门套餐网络。网络运行时，会有一个排序者（称为“SOLO”排序服务）、两个组织、一个 CA 以及每个组织一个同级。此外还会创建一个缺省通道。
6. 单击**启动**按钮。

您可以在 [{{site.data.keyword.Bluemix_notm}} 服务仪表板 ![外部链接图标](images/external_link.svg "外部链接图标")](https://console.bluemix.net/dashboard/services "{{site.data.keyword.Bluemix_notm}} 服务仪表板") 中找到区块链服务实例。


## 邀请成员
{: #inviting-members}

您可以邀请其他[组织](/docs/services/blockchain/glossary.html#organization)作为[成员](/docs/services/blockchain/glossary.html#member)加入入门套餐网络，以便可以相互执行[事务处理](/docs/services/blockchain/glossary.html#transaction)。此外，如果是出于学习和测试目的而使用入门套餐，那么可以通过自行向网络添加成员来模拟多组织网络。

1. 在“网络监视器”的“成员”屏幕中，单击**邀请成员**按钮。
2. 这将打开“邀请成员”窗口。
    - 如果要邀请其他组织，请选择“邀请成员”。指定要邀请的组织的名称和操作员电子邮件地址。还可以在“添加注释”字段中，输入要包含在邀请中的其他信息。单击**发送邀请**按钮。受邀组织将收到一封邀请电子邮件，然后可以遵循电子邮件中的指示信息来加入您的网络。
    - 如果要添加可以添加到通道的其他组织，请选择“添加成员”。为新组织指定名称。可以选择将同级添加到新组织，也可以日后在“网络监视器”中进行添加。单击**创建**按钮。请注意，如果为新组织添加了同级，您需要切换到此新组织才能查看同级。<!--有关切换组织的更多信息，请参阅[切换组织](/docs/services/blockchain/v10_dashboard.html#switch-organizations)。-->


## 加入网络
{: #joining-a-network}

如果您受网络发起者邀请，那么您将收到电子邮件通知，其中包含有关如何加入网络的指示信息。请遵循电子邮件中的指示信息进行操作，完成后您将成为网络中的其中一个成员。

您需要在 {{site.data.keyword.cloud_notm}} 中使用入门套餐成员资格选项来创建 [{{site.data.keyword.blockchain}} Platform 服务实例 ![外部链接图标](images/external_link.svg "外部链接图标")](https://console.bluemix.net/catalog/services/blockchain)。

1. 使用 {{site.data.keyword.cloud_notm}} 帐户登录。如果您没有帐户，请单击**注册以创建**按钮。
2. 选择要存储 {{site.data.keyword.blockchain}} 网络的 Cloud Foundry 组织。
3. 从价格套餐表中选择**入门成员资格套餐**。
4. 单击**创建**按钮。这将打开服务实例页面并弹出欢迎面板。请注意，可以选择加入网络，也可以继续创建自己的网络。要创建网络，请参阅[创建网络](#creating-a-network)中的步骤 4。
5. 在欢迎面板上，选择**加入现有网络**，从下拉列表中选择要加入的网络，然后单击**继续**。

可以在 [{{site.data.keyword.cloud_notm}} 服务仪表板 ![外部链接图标](images/external_link.svg "外部链接图标")](https://console.bluemix.net/dashboard/services "{{site.data.keyword.cloud_notm}} 服务仪表板") 中找到您的区块链服务实例。


## 创建通道
{: #create-channels}

多个组织集合使用通道进行事务处理，而不向其他组织公开数据。您可以使用入门套餐网络的精选成员创建[通道](/docs/services/blockchain/glossary.html#channel)以及可以更新该通道的策略。

有关更多信息，请参阅[创建通道](/docs/services/blockchain/howto/create_channel.html#creating-a-channel)。如果另一个组织邀请您加入某个通道，您还将找到有关如何接受邀请并将您的同级加入到该通道的指示信息。

## 开发链代码
{: #develop-chaincode}

[链代码](/docs/services/blockchain/glossary.html#chaincode)（有时称为智能合同）是允许您在区块链分类帐上读取和更新数据的软件。链代码可以将业务逻辑转变为区块链网络的所有成员同意和验证的可执行程序。

有关更多信息，请参阅[开发链代码](/docs/services/blockchain/howto/develop_chaincode.html)教程以了解如何开始编写链代码，以及了解可通过链代码访问的光纤网功能部件。

## 安装和实例化链代码
{: #install-instantiate-chaincode}
在加入通道和开发业务逻辑后，需要在网络中的同级上安装链代码。您可以使用网络监视器在组织的同级上安装和实例化链代码，以及更新链代码以促进持续开发。

有关部署预构建样本的更多信息，请参阅[安装、实例化和更新链代码](/docs/services/blockchain/howto/install_instantiate_chaincode.html)。


## 使用 Fabric SDK 开发应用程序
{: #develop-apps}

{{site.data.keyword.blockchainfull_notm}} Platform 提供了可使用 Fabric SDK 连接到的 API 端点。您可以使用 SDK 从应用程序调用链代码，并将事务提交到区块链网络。

有关更多信息，请参阅[开发应用程序](/docs/services/blockchain/v10_application.html)教程。


## 使用 Hyperledger Composer 开发和部署定制的业务网络
{: #develop-composer}

入门套餐可以将 {{site.data.keyword.blockchainfull_notm}} Platform: Develop 开发者环境和 Hyperledger Composer 开发者工具集相集成。可以基于您的业务需求来开发区块链网络。开发业务网络后，可以将其部署到入门套餐网络。

有关更多信息，请参阅[开发网络](/docs/services/blockchain/develop.html)和[使用入门套餐部署业务网络](/docs/services/blockchain/develop_starter.html)。


## 部署样本应用程序
{: #sample-applications}

使用入门套餐时，只需单击几下就可将样本应用程序部署到网络中。您可以使用这些样本了解有关链代码和 Hyperledger Composer 业务网络的更多信息，并将它们用作自己的开发的起点。

有关更多信息，请参阅[部署样本应用程序](/docs/services/blockchain/howto/prebuilt_samples.html)。


## 监视网络资源
{: #monitoring-resources}

如果应用程序请求事务处理，您可以在“网络监视器”中查看事务处理状态信息。有关网络监视的更多信息，请参阅[监视网络](/docs/services/blockchain/howto/monitor_network.html)。


## 检索网络凭证和连接概要文件
{: #retrieving-network-credentials}

在 {{site.data.keyword.cloud_notm}} 中创建入门套餐网络后，可以在服务实例页面中或在“网络监视器”中检索网络凭证和连接概要文件。

### 在服务实例页面中进行检索
一旦创建服务实例后，您将位于服务实例页面上。您还可以在 [{{site.data.keyword.cloud_notm}} 服务仪表板 ![外部链接图标](images/external_link.svg "外部链接图标")](https://console.bluemix.net/dashboard/services "{{site.data.keyword.cloud_notm}} 服务仪表板") 中单击服务来打开服务实例页面。

使用以下步骤来检索服务凭证：
1. 在服务实例页面上，单击左侧导航器中的**服务凭证**，以显示“服务凭证”屏幕。
2. 单击“服务凭证”屏幕中的**新建凭证**。
3. 在“添加新凭证”屏幕中，为凭证提供名称，然后在“添加内联配置参数”字段中，输入 **{"type": "service_instance_token"}**。单击**添加**。新的凭证会添加到表中。您可以单击“操作”列下的**查看凭证**以查看凭证详细信息。此凭证包含 API 密钥和私钥，可用于授权 API。


![检索网络凭证](images/service_credentials.gif "检索网络凭证")

### 在网络监视器中进行检索
可以在“网络监视器”的“API”屏幕上找到网络凭证。有关使用 API 的更多信息，请参阅[使用 Swagger API 与网络进行交互](/docs/services/blockchain/howto/swagger_apis.html)。

可以在“网络监视器”的“概述”屏幕上检索连接概要文件。单击“概述”屏幕上的**连接概要文件**按钮，连接概要文件会显示在新页面中。

## 使用 Swagger API 管理网络
{: #swagger}

{{site.data.keyword.blockchainfull_notm}} Platform 公开了使用 Swagger 生成的若干 REST API，可用于管理网络中的节点、通道、同级和成员。应用程序可以使用这些 API 来控制重要的网络资源，而无需使用“网络监视器”。

要了解更多信息，请参阅[使用 Swagger API 与网络进行交互](/docs/services/blockchain/howto/swagger_apis.html)


## 重置网络
{: #resetting-network}

如果要清除定制配置、正在运行的链代码或已部署的应用程序，您可以将网络重新设置为初始缺省配置。有关更多信息，请参阅[重置网络](/docs/services/blockchain/v10_dashboard.html#reset-network)。


## 从入门套餐迁移到企业套餐
{: #migrate}

可以将针对入门套餐网络测试的 `.bna`、链代码和应用程序部署到企业套餐网络中。

如果您有自己的业务网络归档文件 (`.bna`)，请遵循[将业务网络部署到企业套餐](./develop_enterprise.html)的指示信息。如果您没有 `.bna` 文件，请使用 `composer network download` 命令在入门套餐实例中检索该文件。有关 `composer network download` 命令的更多信息，请参阅 [Hyperledger Composer 命令行文档 ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger.github.io/composer/latest/reference/commands){:new_window}。

链代码与 `.bna` 文件类似，也是在外部开发的。要将针对入门套餐网络测试的链代码部署到企业套餐中，请遵循[安装、实例化和更新链代码](/docs/services/blockchain/howto/install_instantiate_chaincode.html#installchaincode)中的指示信息进行操作。

<!--
As you can see in [Deploying sample applications](/docs/services/blockchain/howto/prebuilt_samples.html), Starter Plan makes it easy to get a sample application integrated with your network by using Toolchain. This setup also allows for continuous integration by automatically updating your sample application whenever your forked application repo is changed. If you want to deploy this application into an Enterprise Plan network, you can copy your forked application repo into a new repo and then follow the instructions in [Deploying sample applications manually](howto/prebuilt_samples.html#deploy_sample_applications_manually).
-->

如果在入门套餐网络中部署了任何样本应用程序，但现在希望将此应用程序部署到企业套餐网络中，那么可以将派生的应用程序存储库复制到新的存储库中，然后遵循样本应用程序中的指示信息以在企业套餐网络上进行部署。


## 删除或离开网络
{: #delete-network}

如果您要删除或离开网络，可以从 {{site.data.keyword.cloud_notm}}“仪表板”中删除区块链服务实例。

**注**：在离开网络之前，请确保您不是该网络上的任何通道的成员。否则，当您离开网络时将会获得错误。除去通道成员需要完成通道更新过程。有关通道更新过程的更多信息，请参阅[更新通道](/docs/services/blockchain/howto/create_channel.html#updating-a-channel)。


<!--
## References
* For more information about {{site.data.keyword.blockchainfull_notm}} offerings, see [Blockchain offerings](/docs/services/blockchain/index.html).
* For more information about Hyperledger Fabric V1.2, see [Hyperledger Fabric documentation ![External link icon](images/external_link.svg "External link icon")](http://hyperledger-fabric.readthedocs.io/en/release-1.2/){:new_window}.
-->
