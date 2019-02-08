---

copyright:
  years: 2017
lastupdated: "2017-08-14"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# 样本应用程序

您可以部署和测试样本应用程序，以更好地了解 {{site.data.keyword.blockchainfull}} 网络和应用程序开发。
{:shortdesc}

## 开始之前

请确保在本地文件系统上安装所有必备软件。有关更多信息，请参阅[设置应用程序开发环境](/docs/services/blockchain/v10_application.html#setting-up-application-development-environment)。

您还需要在配置了通道及其同级的 {{site.data.keyword.Bluemix_short}} 上拥有 {{site.data.keyword.blockchain}} 网络。有关更多信息，请参阅[管理网络](/docs/services/blockchain/get_start.html)。当网络准备好可以使用时，请检索应用程序将访问的网络资源的 API 端点。有关更多信息，请参阅[向应用程序添加网络服务凭证](/docs/services/blockchain/v10_application.html#adding-network-service-credentials-to-your-application)。


## Marbles 应用程序

在 Marbles 应用程序中，多个用户可以创建具有不同属性的弹珠，并将其与其他人进行转移。Marbles 应用程序以 JavaScript 编写，而链代码以 Go 编写。

您可以在 [GitHub 中的 Marbles ![外部链接图标](../images/external_link.svg "外部链接图标")](https://github.com/IBM-Blockchain/marbles) 中找到样本代码和指示信息。


## Fabric car 应用程序

在 Fabric car 应用程序中，您可以对分类帐中的汽车记录执行**查询**和**更新**操作。Fabric car 应用程序以 JavaScript 编写，而链代码以 Go 编写。

您可以在 [GitHub 中的 Fabric car ![外部链接图标](../images/external_link.svg "外部链接图标")](https://github.com/hyperledger/fabric-samples/tree/release/fabcar) 中找到样本代码，在[编写第一个应用程序 ![外部链接图标](../images/external_link.svg "外部链接图标")](http://hyperledger-fabric.readthedocs.io/en/latest/write_first_app.html) 中找到指示信息。

<!-- 
## High available application
-->
<!--
The high available application demonstrate how to enable the following features to ensure the high availability of a {{site.data.keyword.blockchain}} network.
1. Have 2 peers and have your application smart enough to talk to one and if it is getting errors or no response switch over to the other.
2. Same for orderers, 2 or 3 and have your application smart enough to fail over if needed.
OR put orderers/peers behind a load balancer.
-->
