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
{:tip: .tip}

# 开发业务网络
{: #develop-the-network}


***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


通过使用 {{site.data.keyword.blockchainfull}} Platform 开发区块链解决方案，可利用 Hyperledger Composer 开放式源代码开发工具集。Hyperledger Composer 使用定制建模语言，该语言与 JavaScript 事务处理和访问控制规则组合使用，可在将任何内容部署到真正的区块链网络_之前_，对区块链业务网络进行完全建模。
{:shortdesc}

**注**：入门套餐提供了您可以学习和试用的样本应用程序。在开发定制业务网络之前，可以先在入门套餐中运行样本应用程序。有关更多信息，请参阅[关于入门套餐](/docs/services/blockchain/starter_plan.html)。

本教程将执行 {{site.data.keyword.blockchain}} 解决方案开发的整个过程，从业务网络创建和建模开始，一直到将其部署到正在运行的区块链网络。

以下步骤概述了 {{site.data.keyword.blockchainfull_notm}} Platform 上托管的 {{site.data.keyword.blockchain}} 解决方案的基本开发过程：

1. 创建数据结构以包含业务网络定义。业务网络使用[建模语言](https://hyperledger.github.io/composer/latest/reference/cto_language)定义资产、参与者和事务等方面。

2. 通过定义部署后将使用的资产类、参与者类、事务类、事件类和访问控制规则，对业务网络进行建模。

3. 将定义的业务网络打包成单个可部署文件。

4. 将打包的业务网络部署到 {{site.data.keyword.blockchainfull_notm}} Platform。一旦部署了业务网络，就可以生成 REST 服务器以允许与外部应用程序集成。

## 开始之前

确保已安装 [{{site.data.keyword.blockchainfull_notm}}: Develop 开发工具](/docs/services/blockchain/develop_install.html)。

## 步骤 1：创建业务网络结构

开发 {{site.data.keyword.blockchain}} 解决方案的一个关键概念是**业务网络定义 (BND)**。BND 为区块链解决方案定义了数据模型、事务处理逻辑和访问控制规则。要创建 BND，您需要在磁盘上创建适用的项目结构。

最简单的入门方法是使用 Yeoman 生成器来创建框架业务网络。Yeoman 生成器会创建一个目录，其中包含业务网络的所有组件，可修改此目录以适合您的特定用例。

1. 为项目创建新目录并在命令行上浏览至此目录。

2. 从新目录，使用 Yeoman 创建框架业务网络。以下命令需要业务网络名称、描述、作者姓名、作者电子邮件地址、许可证选择和名称空间：
    ```
    yo hyperledger-composer:businessnetwork
    ```
    {:codeblock}

3. 输入 `tutorial-network` 作为网络名称，然后输入描述、作者姓名和作者电子邮件的所需信息。

4. 选择 `Apache-2.0` 作为许可证。

5. 选择 `org.acme.biznet` 作为名称空间。

## 步骤 2：定义业务网络

业务网络由资产、参与者、事务、访问控制规则以及（可选）事件和查询组成。在步骤 1 中创建的框架业务网络包含一个模型文件 (`.cto`)，该文件包含业务网络中所有资产、参与者和事务的类定义。框架业务网络还包含一个具有基本访问控制规则的访问控制文档 (`permissions.acl`)、一个包含事务处理器函数的脚本文件 (`logic.js`) 以及一个包含业务网络元数据的包文件 (`package.json`)。

### 对资产、参与者和事务建模

第一个要更新的文档是模型文件 (`.cto`)。此模型文件是使用 [Hyperledger Composer 建模语言](https://hyperledger.github.io/composer/latest/reference/cto_language)编写的。模型文件包含对每类资产、事务、参与者和事件的定义。它隐式扩展了建模语言文档中描述的系统模型。

1. 在选择的编辑器中打开 `org.acme.biznet.cto` 模型文件。其位于在上一步中创建的业务网络的 `models` 文件夹中。

2. 将内容替换为以下信息；

    ```
        /**
         * 我的商品交易网络
         */
        namespace org.acme.biznet
        asset Commodity identified by tradingSymbol {
            o String tradingSymbol
            o String description
            o String mainExchange
            o Double quantity
            --> Trader owner
        }
        participant Trader identified by tradeId {
            o String tradeId
            o String firstName
            o String lastName
        }
        transaction Trade {
            --> Commodity commodity
            --> Trader newOwner
        }
    ```
    {:codeblock}

3. 保存对 `org.acme.biznet.cto` 文件的更改。


### 添加 JavaScript 事务处理逻辑

在模型文件中，定义了 `Trade` 事务，此项指定了与资产和参与者的关系。事务处理器函数文件包含执行模型文件中所定义事务的 JavaScript 逻辑。

`Trade` 事务旨在仅仅接受要交易的 `Commodity` 资产的标识以及要设置为新所有者的 `Trader` 参与者的标识。

1. 打开 `lib` 目录中的 `logic.js` 脚本文件。

2. 将内容替换为以下信息；

    ```
        /**
         * 跟踪商品从一个交易商到另一个交易商的交易
         * @param {org.acme.biznet.Trade} trade - 要处理的交易
         * @transaction
         */
        function tradeCommodity(trade) {
            trade.commodity.owner = trade.newOwner;
            return getAssetRegistry('org.acme.biznet.Commodity')
                .then(function (assetRegistry) {
                    return assetRegistry.update(trade.commodity);
                });
        }
    ```
    {:codeblock}

3. 保存对 `logic.js` 的更改。

### 添加访问控制

访问控制是业务网络的关键部分。所有业务网络都必须有一个名为 `permissions.acl` 的访问控制文件。每当在部署的业务网络中调用任何进程时，都会检查访问控制表，以确保调用身份或参与者可以调用该操作。

在本教程中，请设置简单的 ACL。请注意，此 ACL 不应部署到生产环境。

1. 在 `tutorial-network` 目录中创建 `permissions.acl` 文件。

2. 将以下访问控制规则添加到 `permissions.acl`：

    ```
        /**
         * tutorial-network 的访问控制规则
         */
        rule Default {
            description: "Allow all participants access to all resources"
            participant: "ANY"
            operation: ALL
            resource: "org.acme.biznet.*"
            action: ALLOW
        }

        rule SystemACL {
          description:  "System ACL to permit all access"
          participant: "ANY"
          operation: ALL
          resource: "org.hyperledger.composer.system.**"
          action: ALLOW
        }
    ```
    {:codeblock}

3. 保存对 `permissions.acl` 的更改。

## 步骤 3：打包业务网络

定义业务网络后，必须将其打包成可部署的业务网络归档文件 (`.bna`)。

1. 使用命令行导航至 `tutorial-network` 目录。

2. 在 `tutorial-network` 目录中，运行以下命令：

    ```
composer archive create -t dir -n .
    ```
    {:codeblock}

运行该命令后，将在 `tutorial-network` 目录中创建业务网络归档文件 (`tutorial-network@0.0.1.bna`)。

## 步骤 4：部署业务网络并生成 REST 服务器

使用[入门套餐](./develop_starter.html)或[企业套餐](/docs/services/blockchain/develop_enterprise.html)将业务网络部署到 {{site.data.keyword.blockchainfull_notm}} Platform。

将业务网络部署到 {{site.data.keyword.blockchainfull_notm}} Platform 后，可以生成定制的 REST 服务器。REST 服务器会检查业务网络的内容，并针对网络的资产、事务和参与者构建 RESTful API 调用。[Hyperledger Composer 文档](https://hyperledger.github.io/composer/latest/integrating/getting-started-rest-api)中包含有关生成 REST 服务器的指示信息。
