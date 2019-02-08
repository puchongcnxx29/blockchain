---

copyright:
  years: 2017, 2018
lastupdated: "2018-08-31"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# 管理企业套餐网络
{: #getting-started-with-blockchain}


***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


{{site.data.keyword.blockchainfull}} Platform 企业套餐提供具有很高的安全性、完整性、可扩展性和性能的区块链网络。您可以快速供应功能完整的网络，并使用“网络监视器”（一种 GUI 仪表板）立即运行[链代码](/docs/services/blockchain/glossary.html#chaincode)和应用程序，而无需从头开始设计和配置网络。
{:shortdesc}

**注**：{{site.data.keyword.blockchainfull_notm}} Platform 企业套餐提供的是生产环境。如果您需要开发和测试环境，请参阅[关于入门套餐](/docs/services/blockchain/starter_plan.html)。

本教程介绍了在获取 IBM 的高可用性和安全环境上托管的企业套餐网络时需要遵循的先决条件和步骤。  

以下步骤指示启动具有多个网络[成员](/docs/services/blockchain/glossary.html#member)的企业套餐网络的基本流程：
1. **网络发起者**作为一种特殊类型的网络成员，创建网络并定义管控策略。然后，网络发起者可以邀请其他[组织](/docs/services/blockchain/glossary.html#organization)作为网络成员加入此网络。有关更多信息，请参阅[创建网络](#creating-a-network)。
2. 受邀**网络成员**会收到电子邮件通知，该通知为这些成员提供加入 {{site.data.keyword.blockchain}} 网络的指示信息。除了电子邮件通知中的指示信息外，还可以遵循[加入网络](#joining-a-network)中的步骤。
3. 在创建或加入网络之后，所有**网络成员**都可以进入“网络监视器”来配置和管理其网络资源。您可以设置[通道](/docs/services/blockchain/glossary.html#channel)，使一组网络成员在仅通道成员才能访问的特定于通道的分类帐上执行专用事务处理。在“网络监视器”中，您还可以将自己的同级加入通道，并在这些同级上安装和实例化链代码。有关更多信息，请参阅[配置网络资源和环境](#configuring-network-resources-and-environment)。
4. 开发应用程序后，**应用程序开发者**会启用其应用程序与网络之间的交互。有关更多信息，请参阅[实现应用程序与网络的交互](#enabling-applications-to-interact-with-the-network)。
5. **网络操作员**在“网络监视器”中监视其通道上的事务处理。有关更多信息，请参阅[监视网络资源](#monitoring-network-resources)。


## 创建网络
开始之前，需要在 {{site.data.keyword.Bluemix_notm}} 上创建 [{{site.data.keyword.blockchain}} Platform 服务实例 ![外部链接图标](images/external_link.svg "外部链接图标")](https://console.bluemix.net/catalog/services/blockchain)。您需要使用 {{site.data.keyword.Bluemix_notm}} 标识登录。如果没有标识，请单击**注册以创建**按钮。请为您的实例重命名服务和凭证名称，以便将来易于识别。选择 {{site.data.keyword.Bluemix_notm}} 区域、组织和空间，您可以在其中部署 {{site.data.keyword.blockchain}} 网络。接着，从价格套餐表中选择**企业成员资格套餐**，然后单击**创建**按钮。  

您可以在 [{{site.data.keyword.Bluemix_notm}} 服务仪表板 ![外部链接图标](images/external_link.svg "外部链接图标")](https://console.bluemix.net/dashboard/services "{{site.data.keyword.Bluemix_notm}} 服务仪表板") 中找到 {{site.data.keyword.blockchain}} Platform 服务实例。  

如果您是网络发起者，请单击**创建网络**按钮，以启动 {{site.data.keyword.blockchain}} 网络。请遵循向导，以完成网络和资源的基本配置。  
![“创建网络”向导](images/create_network_name.png "“创建网络”向导")  

1. 在“开始使用”屏幕中，为您的网络提供一个名称，选择 {{site.data.keyword.Bluemix_notm}} 组织的位置，并添加您所在机构的名称。当您邀请其他网络成员时，他们将查找此网络名称以加入。单击**下一步**。
2. （可选）在“邀请成员”屏幕中，输入要邀请其加入网络的成员的机构名称和电子邮件地址。您指定的机构名称不是正式名称。只要机构易于识别，且在加入网络时可进行更改即可。请注意，一个网络最多可以有 15 个成员，包括您自己。此步骤是可选的，您可以稍后在“网络监视器”中邀请成员加入您的网络。单击**下一步**。在完成创建网络的所有步骤后，您邀请的成员将收到有关邀请的电子邮件通知。
3. 在“定义管控规则”屏幕中，为成员资格、通道创建和链代码建立策略。缺省情况下，所有网络成员都可以邀请其他成员加入网络、创建通道和实例化链代码。当前，网络使用的是缺省管控策略。单击**下一步**。
4. 在“复查摘要”屏幕中，验证网络配置。如果要进行修改，请单击该部分标题旁边的**编辑**，或者单击**上一步**按钮以返回到先前屏幕。完成网络配置后，单击**完成**。  
5. 在“已创建网络”屏幕中，您将收到您的网络已成功创建的通知。可以单击**添加[同级](/docs/services/blockchain/glossary.html#peer)**来配置网络资源，或者直接单击**进入监视器**以打开“网络监视器”。您还可以日后在“网络监视器”中添加同级。有关同级的更多信息，请参阅[添加同级](/docs/services/blockchain/v10_dashboard.html#peers)。

现在，您已成功部署能够支持以下网络资源的 {{site.data.keyword.blockchain}} 网络：  
* 成员特定认证中心 (CA)
* 缺省管控策略
* 最多 15 个网络成员  
* 三个订货人和两个中间 CA 节点
* 每个成员最多有三个小同级  
* 崩溃容错订购服务
* 最多 150 个通道
* “网络监视器”中的每个成员最多有 10 个链代码实例化


## 加入网络
与创建网络类似，您需要在 {{site.data.keyword.Bluemix_notm}} 上创建 [{{site.data.keyword.blockchain}} Platform 服务实例 ![外部链接图标](images/external_link.svg "外部链接图标")](https://console.bluemix.net/catalog/services/blockchain)。您需要使用 {{site.data.keyword.Bluemix_notm}} 标识登录。如果没有标识，请单击**注册以创建**按钮。请为您的实例重命名服务和凭证名称，以便将来易于识别。选择 {{site.data.keyword.Bluemix_notm}} 区域、组织和空间，您可以在其中部署 {{site.data.keyword.blockchain}} 网络。接着，从价格套餐表中选择**企业成员资格套餐**，然后单击**创建**按钮。

您可以在 [{{site.data.keyword.Bluemix_notm}} 服务仪表板 ![外部链接图标](images/external_link.svg "外部链接图标")](https://console.bluemix.net/dashboard/services "{{site.data.keyword.Bluemix_notm}} 服务仪表板") 中找到 {{site.data.keyword.blockchain}} Platform 服务实例。

如果您是受邀网络成员，请单击**暂挂邀请 ->** 按钮，从下拉列表中选择要加入的网络，然后单击**加入网络！**按钮。请遵循向导，以查看网络的基本配置并配置自己的网络资源。  
![“加入网络”向导](images/join_network_name.png "“加入网络”向导")  

1. 在“开始使用”屏幕中，输入组织的名称，然后单击**下一步**。
2. 在“复查管控规则”屏幕中，复查网络在成员资格、通道创建和链代码方面的管控策略。单击**下一步**。
3. （可选）在“添加同级”屏幕中，选择要添加的同级数量。单击**下一步**。网络中的每个成员最多可添加 3 个同级。此步骤是可选的，您可以稍后在“网络监视器”中添加同级。有关同级的更多信息，请参阅[添加同级](/docs/services/blockchain/v10_dashboard.html#peers)。
4. 在“复查网络摘要”屏幕中，验证网络配置。如果要进行修改，请单击**上一步**按钮以返回到先前屏幕。完成资源配置后，单击**完成**。系统将通知您已成功加入网络。然后，可以单击**进入监视器**以打开“网络监视器”。

<!-- or click **Create a Channel** to initiate a channel creation request. You can create channels later in the Network Monitor. For more information, see [Channels](/docs/services/blockchain/v10_dashboard.html#channels).  -->


## 配置网络资源和环境

1. 创建或加入 {{site.data.keyword.blockchain}} 网络后进入“网络监视器”。“网络监视器”是一种 GUI 仪表板，您可以在其中管理和跟踪网络状态信息。有关更多信息，请参阅[网络监视器](/docs/services/blockchain/v10_dashboard.html)。
2. 将自己的同级添加到网络中。如果已添加足够多的同级，请跳过此步骤。同级运行链代码，它们是与应用程序交互的端点。单击“概述”屏幕中的**添加同级**，然后选择同级的数量和大小。有关更多信息，请参阅[概述](/docs/services/blockchain/v10_dashboard.html#overview)。
3. 设置通道。同一通道中的所有成员都将供应通道特定的分类帐，该分类帐可提供数据隔离和机密性。有关如何创建通道的更多信息，请参阅[创建通道](/docs/services/blockchain/howto/create_channel.html#creating-a-channel)。  
    如果您是已受邀加入某个通道的通道成员，那么您将收到电子邮件通知，其中包含指向向导的链接，该向导将允许您加入该通道。
4. 将同级加入通道。只有与该通道相关联的同级才能访问其分类帐。有关更多信息，请参阅[通道](/docs/services/blockchain/v10_dashboard.html#channels)。
5. 安装和实例化链代码。所有通道成员都需要在将运行此链代码的每个同级上安装具有相同名称和版本的相同链代码。安装链接代码后，需要在通道上对其进行实例化后方可使用。有关更多信息，请参阅[安装、实例化和更新链代码](/docs/services/blockchain/howto/install_instantiate_chaincode.html)。  

**注**：要实现高可用性，每个组织必须至少购买两个同级，并且在一个通道中，每个参与成员必须至少加入 2 个同级。


## 检索网络凭证和连接概要文件
在 {{site.data.keyword.cloud_notm}} 中创建企业套餐网络后，可以在服务实例页面或在“网络监视器”中检索网络凭证和连接概要文件。

### 在服务实例页面中进行检索
一旦创建服务实例后，您将位于服务实例页面上。您还可以在 [{{site.data.keyword.cloud_notm}} 服务仪表板 ![外部链接图标](images/external_link.svg "外部链接图标")](https://console.bluemix.net/dashboard/services "{{site.data.keyword.cloud_notm}} 服务仪表板") 中单击服务来打开服务实例页面。

使用以下步骤来检索服务凭证：
1. 在服务实例页面上，单击左侧导航器中的**服务凭证**，以显示“服务凭证”屏幕。
2. 单击“服务凭证”屏幕中的**新建凭证**。
3. 在“添加新凭证”屏幕中，为凭证提供名称，然后在“添加内联配置参数”字段中，输入 **{"type": "service_instance_token"}**。单击**添加**，新的凭证会添加到表中。您可以单击“操作”列下的**查看凭证**以查看凭证详细信息。此凭证包含 API 密钥和私钥，可用于授权 API。
    

### 在网络监视器中进行检索
可以在“网络监视器”的“API”屏幕上找到网络凭证。有关使用 API 的更多信息，请参阅[使用 Swagger 试用 API](/docs/services/blockchain/howto/swagger_apis.html)。

可以在“网络监视器”的“概述”屏幕上检索连接概要文件。单击“概述”屏幕上的**连接概要文件**按钮，连接概要文件会显示在新页面中。


## 开发并部署定制业务网络
可以使用 {{site.data.keyword.blockchainfull_notm}} Platform: Develop 开发者环境和 Hyperledger Composer 开发者工具集，基于业务需求来开发业务网络。开发业务网络后，可以将其部署到企业套餐网络。

有关更多信息，请参阅[开发网络](/docs/services/blockchain/develop.html)和[在企业套餐上部署业务网络](/docs/services/blockchain/develop_enterprise.html)。


## 实现应用程序与网络的交互
应用程序会利用 SDK API 与 {{site.data.keyword.blockchain}} 网络资源进行交互。您需要在应用程序中添加网络资源的 API 端点信息，以便应用程序最终可以将同级设定为事务处理请求的目标。然后，可以从“网络监视器”中添加 API 端点信息。可以在本地文件系统或 {{site.data.keyword.Bluemix_notm}} 上托管应用程序。有关更多信息，请参阅[开发应用程序](/docs/services/blockchain/v10_application.html)。

## 监视网络资源  
从应用程序触发事务后，您可以在“网络监视器”中查看事务处理状态信息。有关网络监视的更多信息，请参阅[监视网络](/docs/services/blockchain/howto/monitor_network.html)。

## 离开网络
如果您要离开网络，请从 {{site.data.keyword.Bluemix_notm}} 仪表板删除区块链服务实例。  

**注**：在离开网络之前，请确保您不是网络任何通道中的成员。否则，当您离开网络时将会获得错误。除去通道成员需要完成通道更新过程。有关通道更新过程的更多信息，请参阅[更新通道](/docs/services/blockchain/howto/create_channel.html#updating-a-channel)。


<!--
## References
* For more information about {{site.data.keyword.blockchainfull_notm}} offerings, see [Blockchain offerings](/docs/services/blockchain/index.html).
* For more information about Hyperledger Fabric, see [Hyperledger Fabric documentation ![External link icon](images/external_link.svg "External link icon")](http://hyperledger-fabric.readthedocs.io/en/release-1.1/){:new_window}.
-->
