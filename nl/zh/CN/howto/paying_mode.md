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

# 付费方式
{: #paying-mode}


***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


{{site.data.keyword.blockchainfull}} Platform 按月收取成员资格费用和同级费用。网络成员可以使用自己的 {{site.data.keyword.cloud_notm}} 帐户（其中包含用于创建网络实例的空间）来支付帐单。或者，一个网络成员可以支付网络中其他成员的帐单，甚至可以支付所有成员的帐单为整个网络付费。
如有需要，付款人还可以将帐单传输给其他成员。
{:shortdesc}

根据您选择的网络套餐和使用的资源量，所付费用各不相同。有关定价的更多信息，请参阅[定价](/docs/services/blockchain/pricing.html)。

## 先决条件
{: #prereq}

每个网络成员都必须具有付费 {{site.data.keyword.cloud_notm}} 帐户（例如**现买现付**帐户），这样才能在 {{site.data.keyword.cloud_notm}} 上创建 {{site.data.keyword.blockchainfull_notm}} Platform 服务实例。如果您没有帐户，请[注册](https://console.bluemix.net/registration/)帐户，并将其升级到付费帐户。要将帐户升级为“现买现付”类型，请转至 {{site.data.keyword.cloud_notm}} 控制台中的**管理** > **计费和使用情况** > **计费**，然后单击**添加信用卡**。


## 为自己付费
{: #pay-your-own}

在 {{site.data.keyword.blockchainfull_notm}} Platform 上创建或加入任何区块链网络后，将向您的 {{site.data.keyword.cloud_notm}} 帐户收取成员资格费用和同级费用。


## 为其他网络成员付费
{: #pay-for-others}

网络中的一个成员可以支付同一区块链网络中的一个或多个其他网络成员的费用。要以此方式付款，付款人和其他网络成员必须完成以下步骤。

1. 付款人在 {{site.data.keyword.cloud_notm}} 中为自己要为其付费的成员创建单独的 Cloud Foundry 空间。
   1. 登录到 {{site.data.keyword.cloud_notm}}。
   2. 在菜单栏中，单击**管理** > **帐户** > **Cloud Foundry 组织**。
   3. 单击将在其中创建网络的 Cloud Foundry 组织的**查看详细信息**。如果您没有用于网络的组织，请单击**添加新的 Cloud Foundry 组织**按钮进行创建。
   4. 单击**添加 Cloud Foundry 空间**按钮，为另一个网络成员创建一个空间。为空间选择区域，为其命名，然后单击**添加**。请注意，只有空间创建者才能访问创建的空间。  
   **注**：如果付款人为多个成员付费，那么付款人需要重复此步骤来为付款人将为其付费的每个成员创建单独的空间。
2. 付款人邀请其他成员加入付款人的 {{site.data.keyword.cloud_notm}} 帐户，并向其授予对其特定空间的访问权。确保每个成员只有权访问一个空间。
   1. 在菜单栏中，单击**管理** > **帐户** > **用户**。  
   2. 单击**邀请用户**，然后分配用户访问权：
      1. 指定将邀请加入 Cloud Foundry 组织的单个用户的电子邮件地址。
      2. 在 **Cloud Foundry 访问权**部分中，进行以下选择：
         - **组织**：要创建网络的 Cloud Foundry 组织。
         - **组织角色**：审计员。
         - **区域**：在其中为此用户创建空间的区域。
         - **空间**：为此用户创建的空间。
         - **空间角色**：开发者。
      3. 单击**邀请用户**。  
   **注**：如果付款人为多个成员付费，那么付款人需要重复此步骤以邀请其他成员并为每个成员分配访问权。
3. 付款人在网络的“网络监视器”中创建区块链网络 {{site.data.keyword.blockchainfull_notm}} Platform 并邀请其他成员加入区块链网络。有关邀请网络成员的更多信息，请参阅[成员](https://console.bluemix.net/docs/services/blockchain/v10_dashboard.html#members)。
4. 随后每个成员都会收到一封通知邮件，其中包含加入网络的邀请。成员可以完成以下步骤来加入网络。
   1. 单击通知电子邮件中的“继续”按钮，这会使您转至 {{site.data.keyword.cloud_notm}} 中的区块链服务页面。
   2. 登录到 {{site.data.keyword.cloud_notm}}，并确保使用付款人将您添加到的正确的 {{site.data.keyword.cloud_notm}} 组织和空间。
      1. 在 {{site.data.keyword.cloud_notm}} 控制台中，单击右上角中的个人档案图像。
      2. 从**帐户**下拉列表中选择付款人的帐户。请注意，在其中供应区块链服务实例的帐户和组织将成为计费的对象。  
   3. 在 {{site.data.keyword.cloud_notm}} 目录中，搜索**区块链**并创建服务实例。
      1. 在**服务名称**字段中，输入描述性名称供未来识别。
      2. 验证组织和空间是否为付款人将您添加到的组织和空间。
      3. 选择要用于创建网络实例的成员资格套餐。您的成员资格套餐必须与付款人用于创建网络的成员资格套餐相同。
      4. 单击**创建**。
   4. 创建区块链服务实例后，遵循加入网络的向导。有关更多信息，请参阅[加入网络](https://console.bluemix.net/docs/services/blockchain/get_start.html#joining-a-network)。

### 已知限制
{: #pay-for-others-limitation}
- 由于所有成员均在付款人的 {{site.data.keyword.cloud_notm}} 帐户之内，因此付款人有权访问所有成员的区块链实例并可以对其进行模拟。因此，这种付费方式最好用于 PoC 环境，或者用于付款人处理 {{site.data.keyword.blockchainfull_notm}} Platform 的所有管理工作，并且仅向成员提供一个应用程序的情况。  
- 付款人的 Cloud Foundry 组织中的任何成员都可以查看该组织中的所有空间。但是，他们无法编辑或修改组织，因为他们没有相应的访问权。  
- 在将所有成员添加到付款人的 {{site.data.keyword.cloud_notm}} 帐户，然后为其分配供应区块链实例并加入网络的访问权后，付款人还向成员授予了创建其他服务的访问权，而这可能会产生额外费用。所有成员都可签署一份协议，确信他们不会在其帐户创建创建导致对付款人帐户的额外成本的任何未经授权的服务。  

## 让成员为您付款
{: #let-other-pay}

替代方法是向付款人的帐户添加其他成员的帐户，其他成员也可将付款人的帐户作为**记帐管理员**添加到他们的帐户。要以此方式付款，付款人和其他网络成员必须完成以下步骤。

1. 付款人使用付款人的帐户创建区块链网络 {{site.data.keyword.blockchainfull_notm}} Platform。
2. 其他成员将付款人的帐户作为**记帐管理员**添加到他们的 {{site.data.keyword.cloud_notm}} 帐户。
  1. 在 {{site.data.keyword.cloud_notm}} 控制台顶部的菜单栏中，单击**管理** > **帐户** > **用户**。
  2. 在**用户**页面上，单击**邀请用户**按钮。
  3. 在**电子邮件地址**字段下添加付款人的帐户。
  4. 在 **Cloud Foundry 访问权**部分下，从下拉列表中选择想要将付款人的帐户添加到的组织。
  5. 选择**记帐管理员**作为**组织角色**。
  6. 相应地配置其他字段，然后单击**邀请用户**。  
3. 在网络的“网络监视器”中，付款人邀请其他成员加入区块链网络。有关邀请网络成员的更多信息，请参阅[成员](https://console.bluemix.net/docs/services/blockchain/v10_dashboard.html#members)。
4. 随后每个成员都会收到一封通知邮件，其中包含加入网络的邀请。然后，成员可加入网络。有关更多信息，请参阅[为其他网络成员付费](#pay-for-others)下的**步骤 4**。

### 已知限制
{: #let-other-pay-limitation}

在将付款人的 {{site.data.keyword.cloud_notm}} 帐户作为记帐管理员添加到其他成员帐户并且其他成员使用其帐户购买任何其他服务时，付款人也需要承担这些费用。所有成员都可签署一份协议，确信他们不会在其帐户创建创建导致对付款人帐户的额外成本的任何未经授权的服务。  


## 将帐单传输给其他成员
{: #transfer-billing}

网络成员可以让某个成员为其付费，稍后，其他成员可以接管帐单。您可以从帐户中除去付款人的帐户，从而使付款人停止为您付费。

1. 在 {{site.data.keyword.cloud_notm}} 控制台顶部的菜单栏中，单击**管理** > **帐户** > **Cloud Foundry 组织**。
2. 单击向其添加付款人帐户的组织。
3. 切换到**用户**选项卡。
4. 在用户表中找到付款人的帐户，然后单击**从组织除去**链接。

在从您的帐户中除去付款人的帐户后，将向您的帐户收取区块链网络成员资格费用和同级费用。
