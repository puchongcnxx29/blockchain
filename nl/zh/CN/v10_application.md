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

# 使用 Fabric SDK 开发应用程序
{: #dev_app}


***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


{{site.data.keyword.blockchainfull}} Platform 提供了可用于将应用程序连接到区块链网络的 API。可以使用连接概要文件中的网络 API 端点来调用链代码，并更新或查询同级上特定于通道的分类帐。此外，还可以在 [Swagger UI](/docs/services/blockchain/howto/swagger_apis.html) 中使用 API 来管理节点、通道和网络的成员。
{:shortdesc}

您可以使用本教程来学习如何访问 {{site.data.keyword.blockchainfull_notm}} Platform API，并使用这些 API 向网络注册应用程序。您还将了解如何与网络交互以及从应用程序发出事务。本教程基于 Hyperledger Fabric 文档中的[编写第一个应用程序 ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/write_first_app.html "编写第一个应用程序"){:new_window} 教程。您将用到**编写第一个应用程序**教程中的许多相同文件和命令，但这些文件和命令将用于与 {{site.data.keyword.blockchainfull_notm}} Platform 上的网络进行交互。本教程描述了使用 Hyperledger Fabric Node SDK 进行应用程序开发的每个步骤。您还将了解如何使用 Fabric CA 客户机作为使用 SDK 的替代方法注册和登记用户。


除了本教程外，在创建自己的业务解决方案时，还可以使用 {{site.data.keyword.blockchainfull_notm}} Platform 提供的样本应用程序和链代码作为模板。有关更多信息，请参阅[部署样本应用程序](/docs/services/blockchain/howto/prebuilt_samples.html)。

## 先决条件
您需要满足以下先决条件，才能在 {{site.data.keyword.blockchainfull_notm}} Platform 上使用**编写第一个应用程序**教程。

- 如果 {{site.data.keyword.cloud_notm}} 上没有区块链网络，那么需要使用入门套餐或企业成员资格套餐创建区块链网络。有关更多信息，请参阅[创建入门套餐网络](/docs/services/blockchain/get_start_starter_plan.html#creating-a-network)或[创建企业套餐网络](/docs/services/blockchain/get_start.html#creating-a-network)。

  在进入网络的“网络监视器”后，在“概述”屏幕上至少为组织添加一个同级。然后，在网络中至少创建一个通道。有关更多信息，请参阅[创建通道](/docs/services/blockchain/howto/create_channel.html#creating-a-channel)。**注**：如果使用的是入门套餐网络，那么网络已具有一个名称为 `defaultchannel` 的通道，您可以使用该通道来部署链代码。

- 安装必需的工具以下载 Hyperledger Fabric 样本并使用 Node SDK。
  * [Curl ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/prereqs.html#install-curl "Curl") 或 [Git ![外部链接图标](images/external_link.svg "外部链接图标")](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git "Git"){:new_window}
  * [Node.js ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html#node-js-runtime-and-npm "Node.js"){:new_window}

- 通过下载 `fabric-samples` 目录来安装 Hyperledger Fabric 样本。您可以遵循 Hyperledger Fabric 文档中的[入门指南 ![外部链接图标](images/external_link.svg "外部链接图标 ")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/install.html "入门指南"){:new_window}。

- 导航到本地计算机上的 `fabric-samples` 目录。
  * 使用 `git checkout` 命令来使用与网络 Hyperledger Fabric 版本匹配的分支。可以通过打开“网络监视器”中的[“网络首选项”窗口](/docs/services/blockchain/v10_dashboard.html#network-preferences)来找到 Fabric 版本。
    - 如果您的网络位于 Fabric V1.2 上，您可以使用主要分支。
    - 如果您的网络位于 Fabric V1.1 上，请运行 `git checkout v1.1.0`。
    - 如果您的网络位于 Fabric V1.0 上，请运行 `git checkout v1.0.6`。

  * 导航到 `fabric-samples/fabcar`。您可以复制此目录并对其重命名，以便您可以在干净的目录中尝试测试样本应用程序。

  * 在 `fabcar` 目录中，运行 `npm install` 命令以安装使用 Fabric SDK 所必需的软件包，包括 `fabric-client` 和 `fabric-ca-client`。

- 使用[网络监视器](/docs/services/blockchain/howto/install_instantiate_chaincode.html#installchaincode)在通道上安装和实例化 fabcar 链代码。可以在 `fabric-samples > chaincode > fabcar > go` 文件夹下的 `fabric-samples` 文件夹中找到 fabcar 链代码。

- 在“网络监视器”的“概述”屏幕上检索网络的连接概要文件。将连接概要文件保存到 `fabcar` 目录中，并将其重命名为 `creds.json`。

## 使用 Fabric SDK
{: #using-the-fabric-sdks}

Hyperledger Fabric SDK 提供了一组功能强大的 API，支持应用程序与区块链网络进行交互。您可以在 [Hyperledger Fabric SDK 社区文档 ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/getting_started.html#hyperledger-fabric-sdks "Hyperledger Fabric SDK 社区文档"){:new_window} 中找到受支持语言的最新列表。建议将 Node SDK 或 Java SDK 用于 {{site.data.keyword.blockchainfull_notm}} Platform。您可以了解有关这些 SDK 在其各自存储库中提供的 API 的更多信息。

本教程使用 [Node SDK ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/ "Node SDK"){:new_window} 来注册应用程序，然后使用应用程序通过调用和查询链代码来发出事务。本教程描述了需要向 SDK 提供的信息，以便应用程序可以连接到区块链网络。本教程还介绍了一些可以使用的 API，以及 SDK 如何与区块链网络进行交互并向区块链网络提交事务。

## 向应用程序添加网络 API 端点
{: #api-endpoints}

您需要在 {{site.data.keyword.cloud_notm}} 上的区块链网络中，向应用程序提供特定网络资源的 API 端点，包括排序者、CA 和同级节点。应用程序可以通过这些 API 端点与网络进行交互。您可以在网络的连接概要文件中找到 API 端点。连接概要文件为 JSON 格式，包含用于网络资源的 API 端点信息和注册标识/私钥。

1. 使用下列其中一种方法从“网络监视器”中检索网络资源的 API 端点信息：
  * 在“概述”屏幕中，单击**连接概要文件**。连接概要文件包含所有网络资源的一组完整 API 端点的信息。![“网络监视器”中的连接概要文件](images/service_credentials.png "“网络监视器”中的连接概要文件")

  * 如果已在网络中运行链代码，那么可以获取特定于链代码的 API 端点信息。在“通道”屏幕上，单击正在运行链代码的通道行以打开特定通道屏幕。然后找到链代码并单击 **JSON** 按钮。
    ![每个链代码的 API 端点](images/channel_chaincode.png "每个链代码的 API 端点")

2. 找到网络资源的 API 端点信息，这类似于以下示例中 `peer1-org1` 行的 URL：
  ```
"peers": {
            "org1-peer1": {
            "url": "grpcs://n7413e3b503174a58b112d30f3af55016-org1-peer1.us3.blockchain.ibm.com:31002",
            "eventUrl": "grpcs://n7413e3b503174a58b112d30f3af55016-org1-peer1.us3.blockchain.ibm.com:31003",
                  ...
  ```

  **注**：您可能希望将组织外部的网络资源设定为应用程序的目标。例如，如果链代码[背书策略](/docs/services/blockchain/howto/install_instantiate_chaincode.html#endorsement-policy)需要来自通道上其他组织的背书，那么您需要获取其同级的端点信息和附带的 TLS 证书。可以在连接概要文件的 peers 部分中找到这些信息。但是，您需要联系其他组织的管理员，了解哪些同级已加入特定通道。

3. 将 API 端点信息插入应用程序的配置文件，如以下示例中所示：
  ```
  grpcs://n7413e3b503174a58b112d30f3af55016-orderer.us3.blockchain.ibm.com:31001
  ```

  您还可以将 [HEAD 请求](/docs/services/blockchain/howto/monitor_network.html#monitor-nodes)发送到这些端点，以检查网络资源的可用性。

  如果使用的是 Fabric SDK，那么还可以使用连接概要文件来连接到网络。本教程提供了手动将 SDK 连接到网络的端点信息。但是，您可以在后面的部分中找到有关[将连接概要文件与 SDK 配合使用](#using-your-connection-profile-with-the-sdk)的教程和指南。

## 注册应用程序
{: #enroll-app}

在将应用程序连接到 {{site.data.keyword.blockchainfull_notm}} Platform 上的网络之前，需要向网络证明应用程序的真实性。我们不会深入探讨 x509 证书和公共密钥基础架构的详细信息，但您可以通过
访问[管理 {{site.data.keyword.blockchainfull_notm}} Platform 上的证书](/docs/services/blockchain/certificates.html)教程来了解更多相关信息。简单来说，Fabric 中的通信流在每个接触点都使用签名/验证操作。因此，任何将调用（例如，分类帐查询或更新）发送到网络的应用程序都需要使用自己的专用密钥对有效内容签名，并附加正确签名的 x509 证书，以用于验证目的。**注册**是从相应的认证中心生成必需的密钥和证书的过程。注册之后，应用程序就可以随时与网络通信了。

本部分说明了如何通过样本代码（**编写第一个应用程序**教程中的一部分内容）检索 Fabric Node SDK 的密钥和证书。您只能使用向认证中心注册的身份来生成证书。以下教程首先使用已向您的 CA 注册的管理员身份进行登记。然后，使用这些证书注册新客户机身份。此教程使用新身份再次登记，并使用这些证书将事务提交到网络<!---You can find an illustration of how the developing applications tutorial interacts with your organization CA in the diagram below.--->

您也可以使用“网络监视器”的“认证中心”屏幕来生成证书，并使用这些证书与网络交互。要了解其方式，请访问[使用网络监视器生成证书](#enroll-panel)。通过[管理证书](/docs/services/blockchain/certificates.html)教程，您还可以了解如何从命令行使用 [Fabric CA 客户机](/docs/services/blockchain/certificates.html#enroll-register-caclient)以生成证书并注册用户。

### 使用 Fabric SDK 进行注册
{: #enroll-app-sdk}

在文本编辑器中，打开 `fabric-samples` 文件夹中 `fabcar` 目录下的 `enrollAdmin.js` 文件。

1. 该文件会首先创建 Fabric 客户机的实例。
  ```
  var fabric_client = new Fabric_Client();
  ```
  {:codeblock}

2. 然后，该文件将创建键值存储 (KVS) 来管理证书。SDK 使用 [KeyValueStore ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/module-api.KeyValueStore.html "KeyValueStore"){:new_window} 类来创建键值存储，并使用 [CryptoSuite ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/module-api.CryptoSuite.html "CryptoSuite"){:new_window} 类来执行加密计算。您可以在下面看到相关代码块。
  ```
  # 创建键值存储，如 fabric-client/config/default.json 的“key-value-store”设置中所定义
  Fabric_Client.newDefaultKeyValueStore({ path: store_path
    }).then((state_store) => {
   // 将该存储分配给 Fabric 客户机
   fabric_client.setStateStore(state_store);
   var crypto_suite = Fabric_Client.newCryptoSuite();
   // 对状态存储（保存用户证书的位置）和
   // 加密存储（保存用户密钥的位置）使用相同的位置
   var crypto_store = Fabric_Client.newCryptoKeyStore({path: store_path});
   crypto_suite.setCryptoKeyStore(crypto_store);
   fabric_client.setCryptoSuite(crypto_suite);
   var	tlsOptions = {
     trustedRoots: [],
     verify: false
   };
  ```
  {:codeblock}

3. 在 KVS 的定义之后，可以使用 [Fabric 客户机 ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/Client.html "Fabric 客户机"){:new_window} 类和 Fabric-CA-Client API <!---[FabricCAServices ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/FabricCAServices.html "FabricCAServices")---> 中的一些方法与 CA 服务器通信。您将需要向 SDK 提供认证中心的名称和 URL。在“网络监视器”的**概述**屏幕中打开**连接概要文件** JSON 文件，并在 `certificateAuthorites` 部分下找到以下变量：
  * CA 的 URL：``certificateAuthorities`` 下的 ``url``
  * 管理用户标识：``enrollId``
  * 管理密码：``enrollSecret``
  * CA 名称：`caName`

  通过以下方式，使用这些信息来**编辑** `enrollAdmin.js` 文件中的相关行：
  ```
  fabric_ca_client = new Fabric_CA_Client('https://<enrollID>:<enrollSecret>@<ca_url_with_port>', null ,<caName>, crypto_suite);
  ```
  {:codeblock}

  例如：
  ```
  fabric_ca_client = new Fabric_CA_Client('https://admin:dda0c53f7b@n7413e3b503174a58b112d30f3af55016-org1-ca.us3.blockchain.ibm.com:31011', null ,'org1CA', crypto_suite);
  ```

  然后，Fabric 客户机将检查应用程序是否已注册。使用连接概要文件中的 `enrollID` 来**编辑**以下行：
  ```
  return fabric_client.getUserContext('<enrollID>', true);
  ```
  {:codeblock}

4. 您需要向 CA 服务器发送“enroll”调用。您的 `admin` 已向网络进行注册。enroll 调用会检索包装在 x509 证书中并由目标 CA 签名的专用密钥和公用密钥。此包装并签名的证书称为 signCert。signCert 允许网络成员验证源自客户机的调用。您需要通过凭证文件提供组织名称、密码和 MSP 标识。在网络凭证的 `certificateAuthorities` 部分中，找到 `enrollID`、`enrollSecret` 和 `x-mspid`。使用这些值**编辑**下面的代码块，然后替换文件中的相关部分。
  ```
  return fabric_ca_client.enroll({
    enrollmentID: '<enrollID>',
    enrollmentSecret: '<enrollSecret>'
      }).then((enrollment) => {
    console.log('Successfully enrolled admin user');
    return fabric_client.createUser(
         username: 'admin',
            mspid: '<x-mspid>',
            cryptoContent: { privateKeyPEM: enrollment.key.toBytes(), signedCertPEM: enrollment.certificate }
        });
  ```
  {:codeblock}

5. 保存 `enrollAdmin.js` 文件。

在 `fabcar` 目录中，通过发出以下命令来注册 admin：
```
node enrollAdmin.js
```
{:codeblock}

注册命令会生成 signCert，并将其导出到名为 `hfc-key-store` 的文件夹。本教程中的未来文件将在此文件夹中查找证书。如果可以在 `hfc-key-store` 文件夹中找到 admin 证书，说明 enroll 命令正常运行。

如果要[使用 SDK 操作网络](#operate-sdk)，那么需要将 admin signCert 上传到 {{site.data.keyword.blockchainfull_notm}} Platform。您可以在 `hfc-key-store` 文件夹中找到 admin signCert。打开 `admin` 文件，然后复制 `certificate` 字段后的引号内的证书。使用工具或文本编辑器将证书转换为 PEM 格式。然后，您可以从“网络监视器”将管理证书上传到区块链网络。有关添加证书的更多信息，请参阅“网络监视器”中[“成员”屏幕的“证书”选项卡](v10_dashboard.html#members)。如果要仅使用 SDK 来调用或查询链代码，那么不必执行此操作。

## 注册应用程序
{: #register-app}

生成客户端证书之后，需要向网络认证中心注册应用程序。注册会将应用程序添加到网络可以识别的组件的列表中。最佳做法是以单独的身份注册应用程序，而不使用 `admin` 来对请求签名。

### 使用 SDK 进行注册
{: #register-app-sdk}

可以使用 `registerUser.js` 文件通过 `admin` signCert 将应用程序注册为 `user1`。在文本编辑器中打开 `registerUser.js`。

1. 向 Fabric CA 客户机的新实例提供 CA URL 和名称。

2. 向 [Fabric Client 类 ![外部链接图标](images/external_link.svg "外部链接图标") 中的 `getUserContext` 方法提供 enrollID，](https://fabric-sdk-node.github.io/Client.html "Fabric Client 类"){:new_window}以检查您的 `admin` 是否已注册并被允许发出此请求。基于下面的样本，在文件中**编辑**相关代码块：

  ```
  // CA 在启用了 TLS 的情况下运行时，请确保将 http 更改为 https
  fabric_ca_client = new Fabric_CA_Client('https://admin:dda0c53f7b@n7413e3b503174a58b112d30f3af55016-org1-ca.us3.blockchain.ibm.com:31011', null , '<caName>', crypto_suite);

  // 首先检查 admin 是否已注册
  return fabric_client.getUserContext('admin', true);
  }).then((user_from_store) => {
  if (user_from_store && user_from_store.isEnrolled()) {
      console.log('Successfully loaded admin from persistence');
      admin_user = user_from_store;
  } else {
      throw new Error('Failed to get admin.... run enrollAdmin.js');
  }
  ```
  {:codeblock}

3. 使用 **Fabric CA 客户机**向 CA 注册和登记用户，然后使用 **Fabric 客户机**创建新的 signCert。使用 MSP 标识和组织亲缘关系来**编辑**以下块。您可以在网络凭证的 certificate authorities 部分中找到 `x-affiliations`，并使用列出的任何亲缘关系。添加要创建的用户的名称。缺省情况下，fabcar 样本使用 `user1`。
  ```
  return fabric_ca_client.register({enrollmentID: 'user1', affiliation: '<x-affiliations>',role: 'client'}, admin_user)
    ...
  return fabric_ca_client.enroll({enrollmentID: 'user1', enrollmentSecret: secret});
  }).then((enrollment) => {
  console.log('Successfully enrolled member user "user1" ');
  return fabric_client.createUser(
   {username: 'user1',
   mspid: '<x-mspid>',
   cryptoContent: { privateKeyPEM: enrollment.key.toBytes(), signedCertPEM: enrollment.certificate }
   });
  ```
  {:codeblock}

4. 保存 `registerUser.js` 文件。

运行 `node registerUser.js` 命令以注册 `user1`。如果可以在 `hfc-key-store` 文件夹中找到 `user1` 证书，说明该命令正常运行。您只能注册一次身份。如果遇到问题，请尝试使用新的用户名来运行 `registerUser.js`。

### 使用网络监视器进行注册

或者，可以使用“网络监视器”的**认证中心**选项卡来注册客户机应用程序。有关更多指示信息，请参阅此[信息](/docs/services/blockchain/v10_dashboard.html#ca)。

## 通过调用和查询链代码来发出事务
{: #invoke-query}

应用程序需要与整个区块链网络交互以提交事务。

1. 应用程序发送要由通道上的同级背书的事务建议。
2. 背书同级将已背书事务返回给应用程序。
3. 应用程序将已背书事务发送给排序服务，以用于将该事务添加到分类帐。

有关完整事务流的更多信息，请参阅 Hyperledger Fabric 文档中的[事务流 ![外部链接图标](images/external_link.svg "外部链接图标")]( https://hyperledger-fabric.readthedocs.io/en/release-1.2/txflow.html "事务流"){:new_window}。开始学习本教程后，请访问[应用程序连接和可用性](#app-connectivity-availability)部分，以获取有关管理 SDK 如何与网络进行交互的提示。

以下样本演示了 Node SDK 如何设置网络拓扑，定义事务建议，然后将事务提交到网络。可以使用 `invoke.js` 文件来调用 `fabcar` 链代码中的函数。通过这些函数，可以在区块链分类帐上创建和转移资产。本教程使用 `initLedger` 函数向通道添加新数据，然后使用 `query.js` 文件来查询数据。

### 调用链代码
{: #invoke}

在文本编辑器中打开 `invoke.js` 文件。

1. 将 `var creds = require('./creds.json')` 添加到该文件的顶部。此代码行允许 `invoke.js` 文件从 `creds.json` 凭证文件中读取信息。

2. 使用 [Fabric 客户机 ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/Client.html "Fabric 客户机"){:new_window} 类通过网络资源的 API 端点来设置 Fabric 网络。此步骤定义客户机将向其提交建议的通道和同级，以及 SDK 随后将向其发送已背书事务的排序服务。**编辑**下面的相关代码块。`creds.peers["org1-peer1"].url` 行从凭证文件导入同级 URL。
  ```
  # 设置 Fabric 网络
  var channel = fabric_client.newChannel('defaultchannel');
  var peer = fabric_client.newPeer(creds.peers["org1-peer1"].url, { pem: creds.peers["org1-peer1"].tlsCACerts.pem , 'ssl-target-name-override': null});
  channel.addPeer(peer);
  var order = fabric_client.newOrderer(creds.orderers.orderer.url, { pem: creds.orderers.orderer.tlsCACerts.pem , 'ssl-target-name-override': null})
  channel.addOrderer(order);
  ```
  {:codeblock}

  新的同级和排序节点变量会打开与区块链网络的 GRPC 连接。有关管理这些连接的更多信息，请参阅[打开和关闭网络连接](#connections)。

  将同级 URL 添加到 `fabric_client.newPeer` 方法时，还可使用下面的代码片段从连接概要文件导入相关的 TLS 证书。在添加排序服务 URL 时，执行了相同的操作。您需要使用这些 TLS 证书来认证与网络的通信。
  ```
  { pem: creds.peers["org1-peer1"].tlsCACerts.pem , 'ssl-target-name-override': null}
  ```
  {:codeblock}

  如果背书策略要求事务由通道中的其他组织进行背书，那么需要在设置网络时使用 `newPeer()` 和 `channel.addPeer()` 方法来添加这些同级组织。组织会向您发送他们已加入特定通道的同级的列表。连接概要文件中将提供端点信息和 TLS 证书。然后，SDK 会将事务发送给添加到通道的所有同级。

  在[使应用程序高度可用](#ha-app)中，还可以添加属于您的组织并已加入通道的其他同级。这将为 SDK 提供其中一个同级中断时执行故障转移的功能。

3. 设置 Fabric 网络并在注册步骤中导入应用程序身份和 signCert 之后，`invoke.js` 文件会定义将提交给网络的建议。可以使用 `fabcar` 链代码中的 `initLedger` 函数向分类帐添加一些初始数据。还可以编辑代码块以调用可在 `fabcar` 链代码中找到的其他函数。
  ```
  var request = {
    //目标：允许缺省值用于分配给客户机的同级
    chaincodeId: 'fabcar',
    fcn: 'initLedger',
    args: [''],
    chainId: 'mychannel',
    txId: tx_id
  };
  ```
  {:codeblock}

  定义请求后，可以将[事务建议 ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/Channel.html#sendTransactionProposal "sendTransactionProposal") 发送给通道上的同级。从同级返回建议后，可以向排序服务[发送事务 ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/Channel.html#sendTransaction "sendTransaction")。

4. 可以添加事件服务来提高事务流的效率。请**编辑**以下部分：
  ```
  let event_hub = fabric_client.newEventHub();
  event_hub.setPeerAddr(creds.peers["org1-peer1"].eventUrl, { pem: creds.peers["org1-peer1"].tlsCACerts.pem , 'ssl-target-name-override': null});
  ```
  {:codeblock}

  虽然样本使用的是基于同级的事件服务，但您应该使用基于通道的侦听器。您可以在[管理事务](#managing-transactions)部分和 [Node SDK 文档 ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/tutorial-channel-events.html "基于通道的事件服务"){:new_window} 中了解更多信息。

5. 缺省情况下，`invoke.js` 以 `user1` 身份提交事务。如果注册了其他名称，那么可以编辑 `invoke.js` 文件。

完成文件编辑后，发出 `node invokke.js` 命令以将事务提交到网络。您应该会看到以下内容，这是对成功调用的确认。
```
Successfully sent Proposal and received ProposalResponse: Status - 200, message - "OK"
```

这指示应用程序已成功调用链代码并将数据添加到分类帐。

### 查询链代码
{: #query}

现在，您可以使用 `query.js` 来读取分类帐。在文本编辑器中打开 `query.js` 文件。

1. 将 `var creds = require('./creds.json')` 添加到该文件的顶部。
2. 使用同级的通道名称和端点信息更新该文件。由于此操作仅读取存储在同级上的数据，因此无需添加排序服务的端点信息。`query.js` 还假定您以 `user1` 身份发送建议。
  ```
  var channel = fabric_client.newChannel('mychannel');
  var peer = fabric_client.newPeer(creds.peers["org1-peer1"].url, { pem: creds.peers["org1-peer1"].tlsCACerts.pem , 'ssl-target-name-override': null});
  channel.addPeer(peer);
  ```
  {:codeblock}

完成编辑文件后，发出 `node query.js` 命令，您应该会看到分类帐上的汽车列表，这类似于以下示例。
```
Query has completed, checking results
Response is  
  [{"Key":"CAR0", "Record":{"colour":"blue","make":"Toyota","model":"Prius","owner":"Tomoko"}},
  {"Key":"CAR1", "Record":{"colour":"red","make":"Ford","model":"Mustang","owner":"Brad"}},
  ...
```
{:codeblock}

有关 fabcar 应用程序及其使用的函数的更多信息，可以访问 Hyperledger Fabric 文档中的完整教程[编写第一个应用程序 ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/write_first_app.html "编写第一个应用程序"){:new_window}。

## 将连接概要文件与 SDK 配合使用
{: #using-your-connection-profile-with-the-sdk}

您可以使用“网络监视器”的**概述**屏幕中的**连接概要文件**使 SDK 连接到网络，而不手动导入网络的端点信息。这简化了连接到认证中心以进行注册的过程。此外，也无需在提交事务之前定义 Fabric 网络。SDK 将直接在连接概要文件中找到相关通道上的同级和排序者。您可以在 [Node SDK 文档 ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/tutorial-network-config.html "连接概要文件教程"){:new_window} 中找到有关如何使用连接概要文件的更多信息。

我们将 `invoke.js` 文件用作示例来查看使用连接概要文件（而不是手动端点）的效率。可以使用 `loadFromConfig` 类来建立 Fabric 客户机的新实例。将 `var fabric_client = new Fabric_Client();` 替换为以下代码。
```
var fabric_client = Fabric_Client.loadFromConfig(path.join(__dirname, './connection-profile.json'));
```
{:codeblock}

可以使用以下行来创建新的通道，而不通过创建同级和排序者，然后将它们添加到通道来设置 Fabric 网络。

```
var channel = fabric_client.newChannel('defaultchannel');
```
{:codeblock}

然后，SDK 将添加使用连接概要文件在通道上定义的同级和排序服务。这将提高编写应用程序的效率，让您在网络成员连接、离开和启动新通道时，更轻松地更新应用程序。请参阅 Node SDK 文档中的[连接概要文件教程 ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/tutorial-network-config.html "连接概要文件教程"){:new_window}，以了解有关所涉及的其他步骤的信息。可以使用此[版本的 fabcar 教程 ![外部链接图标](images/external_link.svg "外部链接图标")](https://www.ibm.com/developerworks/cloud/library/cl-deploy-fabcar-sample-application-ibm-blockchain-starter-plan/index.html){:new_window}，该教程使用的是连接概要文件，而不是手动端点连接。

通过编辑连接概要文件，可以将事务发送给组织外部的同级进行背书。连接概要文件已包含来自 {{site.data.keyword.blockchainfull_notm}} Platform 网络上其他组织中同级的端点信息和 TLS 证书。将同级的名称添加到配置文件的 `channels` 部分中的相关通道，以将同级添加到通道。

## 使用网络监视器生成证书
{: #enroll-panel}

您可以通过“网络监视器”使用管理员身份生成证书，然后将这些证书直接传递到 SDK。这意味着您可以快速开始与网络交互，而不必使用 SDK 生成证书。

浏览至“网络监视器”的“认证中心”面板。单击您的管理员身份旁边的**生成证书**按钮，以从您的 CA 获取新的 signCert 和专用密钥。**证书**字段包含 signCert，刚好位于**专用密钥**上方。您可以单击每个字段末尾的复制图标以复制值。将这些证书保存在您可以将其提供给应用程序的位置。**请注意**，{{site.data.keyword.blockchainfull_notm}} Platform 不存储这些证书。您需要安全地保存并存储它们。

signCert 和专用密钥足以构成用户上下文，该上下文可以对 Node SDK 中的请求进行签名。使用 Client 类的 [createUser ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/Client.html#createUser__anchor "创建用户"){:new_window} 方法创建用户上下文对象。在 `creatUser` 方法中，将身份名称和 mspid 传递给
[user ![外部链接图标](images/external_link.svg "外部链接图标 ")](https://fabric-sdk-node.github.io/global.html#UserOpts "user"){:new_window} 对象，将专有密钥和 signCert 的路径传递给 [cryptoContent ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/global.html#CryptoContent "加密内容"){:new_window} 对象。

作为示例，我们可以使用“认证中心”面板和 `createUser` 类作为开发应用程序教程的一部分。如果您已完成本教程，并且随后已安装并实例化 `fabcar` 链代码，并将一些数据添加到分类帐中。我们可以使用这些证书以 `admin` 用户身份查询分类帐。如果尚未使用上述“网络监视器”生成证书，请遵循指示信息完成此操作。

将专用密钥保存为 privateKey.pem，将 signCert 保存为 certificate.pem，并存储在 `query.js` 所在的相同目录中。在文本编辑器中打开 `query.js`。将以下行添加到文件的顶部：
```
var fs = require('fs');
```
从持久状态导入用户上下文，将以下行
```
return fabric_client.getUserContext('user1', true);
```
替换为使用 signCert 和专用密钥创建新用户上下文的以下代码。
```
return fabric_client.createUser({
		username: 'admin',
		mspid:  'org1',
		cryptoContent: {
			privateKeyPEM: fs.readFileSync(path.join(__dirname,'./privateKey.pem')),
			signedCertPEM: fs.readFileSync(path.join(__dirname,'./certificate.pem'))
		}});
```
以上片段将证书作为 PEM 文件直接读取到 `cryptoContent` 类。由于证书是使用 `admin` 身份生成的，因此用户名将为 `admin`。您可以在连接概要文件的 `certificateAuthorites` 部分中找到 mspid。保存文件并发出命令 `node query.js`。如果成功，查询将返回与之前相同的结果。

## 应用程序连接和可用性的最佳实践
{: #app-connectivity-availability}

Hyperledger Fabric [事务流 ![外部链接图标](images/external_link.svg "外部链接图标")]( https://hyperledger-fabric.readthedocs.io/en/release-1.2/txflow.html "事务流"){:new_window} 跨多个组件，其中客户机应用程序扮演唯一角色。SDK 会将事务建议提交给同级进行背书。然后，SDK 会收集已背书建议以发送给排序服务，然后将事务区块发送给同级以添加到通道分类帐。生产应用程序的开发者应该做好准备来管理 SDK 与其网络之间的交互，以实现效率和可用性。

### 管理事务
{: #managing-transactions}

应用程序客户机应确保验证其事务建议，并确保建议成功完成。建议可能会由于多种原因（例如，网络中断或组件故障）而延迟或丢失。您应该将应用程序准备为可实现[高可用性](#ha-app)，以便处理组件故障。此外，还可以在应用程序中[增大超时值](#set-timeout-in-sdk)，以防止建议在网络可以响应之前超时。

如果链代码未运行，那么发送到此链代码的第一个事务建议将启动链代码。链代码正在启动时，将拒绝其他所有建议，并返回指示链代码当前正在启动的错误。这不同于事务失效。如果在启动链代码期间拒绝了任何建议，那么应用程序客户机需要在链代码启动后再次发送被拒绝的建议。应用程序客户机可以使用消息队列来避免丢失事务建议。

可以使用基于通道的事件服务来监视事务并构建消息队列。[channelEventHub ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/ChannelEventHub.html "channelEventHub"){:new_window} 类可以根据事务、区块和链代码事件来注册侦听器。来自通道 eventhub 的基于通道的侦听器可以扩展到多个通道，并区分不同通道上的流量。

建议您使用 channelEventHub，而不是旧的 eventHub 类。eventHub 采用单个线程，包含所有通道的事件，可以减缓甚至挂起各通道中的侦听器。eventHub 类也不保证会传递事件，不提供从特定点（例如块号）检索事件以跟踪缺失事件的方式。

**请注意**，在未来的 Fabric SDK 发行版中，不推荐使用同级 eventhub。如果您的现有应用程序使用同级 eventhub，那么请更新应用程序以使用通道 eventhub。有关更多信息，请参阅 Node SDK 文档中的
[如何使用基于通道的事件服务 ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/tutorial-channel-events.html "如何使用基于通道的事件服务"){:new_window}。

### 打开和关闭网络连接
{: #connections}

在提交事务建议之前使用 SDK 创建同级和排序节点对象时，您将打开应用程序和网络组件之间的 gRPC 连接。例如，以下命令打开与 `org1-peer1` 的连接。应用程序正在运行时，此连接继续保持活动状态。

```
var peer = fabric_client.newPeer(creds.peers["org1-peer1"].url, { pem: creds.peers["org1-peer1"].tlsCACerts.pem , 'ssl-target-name-override': null});
```
{:codeblock}

管理应用程序和网络之间的连接时，您可能考虑以下建议。

- 与网络交互时，请复用同级和排序节点对象，而不是打开新连接来提交事务。复用同级和排序节点对象可以节省资源并提高性能。  
- 要维护与网络组件的持续连接，请使用 [gRPC 保持活动 ![外部链接图标](images/external_link.svg "外部链接图标 ")](https://github.com/grpc/grpc/blob/master/doc/keepalive.md "gRPC 保持活动")。保持活动让 gRPC 连接保持活动状态，并阻止关闭“未使用的”连接。以下同级连接示例将 gRPC 选项添加到[连接选项 ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/global.html#ConnectionOpts "连接") 对象。gRPC 选项设置为 {{site.data.keyword.blockchainfull_notm}}Platform 建议的值。  
  ```
  var peer = fabric_client.newPeer(creds.peers["org1-peer1"].url, { pem: creds.peers["org1-peer1"].tlsCACerts.pem , 'ssl-target-name-override': null},
  "grpcOptions": {
    "grpc.keepalive_time_ms": 120000,
    "grpc.http2.min_time_between_pings_ms": 120000,
    "grpc.keepalive_timeout_ms": 20000,
    "grpc.http2.max_pings_without_data": 0,
    "grpc.keepalive_permit_without_calls": 1
    }
  );
  ```
  {:codeblock}

  您还可以在网络连接概要文件的 `"peers"` 部分中找到带有所建议设置的这些变量。如果您使用 [SDK 的连接概要文件](#using-your-connection-profile-with-the-sdk)连接到网络端点，那么建议的选项将自动导入到应用程序中。

- 如果不再需要连接，请使用 `peer.close()` 和 `order.close()` 命令来释放资源并防止性能下降。有关更多信息，请参阅 Node SDK 文档中的
[peer close ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/Peer.html#close__anchor "peer close") 和 [orderer close ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/Orderer.html#close__anchor "orderer close") 类。如果您已使用连接概要文件将同级和排序节点添加到通道对象，那么可以使用 `channel.close()` 命令来关闭分配给该通道的所有连接。

### 高可用性应用程序
{: #ha-app}

作为高可用性最佳实践，强烈建议您每个组织至少部署两个同级以实现故障转移。您还需要调整应用程序以获得高可用性。在两个同级上安装链代码，并将这两个同级添加到通道。然后，在设置网络并构建同级目标列表时，请准备好向这两个同级端点[提交事务建议](#invoke)。企业套餐网络有多个排序节点以用于故障转移，允许您的客户机应用程序在一个排序节点不可用时将背书的事务发送到其他排序节点。如果您改为使用[连接概要文件](#using-your-connection-profile-with-the-sdk)来手动添加网络端点，请确保您的概要文件是最新的，并且其他同级和排序节点已添加到概要文件的 `channels` 部分中的相关通道。然后，SDK 可以使用连接概要文件添加通道上加入的组件。

## 启用双向 TLS
{: #mutual-tls}

如果运行的是 Fabric V1.1 级别的企业套餐网络，那么可以选择为应用程序[启用双向 TLS](/docs/services/blockchain/v10_dashboard.html#network-preferences)。如果启用双向 TLS，那么需要更新应用程序以支持此功能。否则，应用程序无法与网络通信。

在连接概要文件中，找到 `certificateAuthorities` 部分，您可在其中找到注册以及获取证书以使用双向 TLS 与网络进行通信所需的以下属性。

- `url`：用于连接到可提供双向 TLS 证书的 CA 的 URL
- `enrollId`：用于获取证书的注册标识
- `enrollSecret`：用于获取证书的注册密钥
- `x-tlsCAName`：用于获取将允许应用程序使用双向 TLS 进行通信的证书的 CA 名称。

有关更新应用程序以支持双向 TLS 的更多信息，请参阅[如何配置双向 TLS ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/tutorial-mutual-tls.html "双向 TLS"){:new_window}。

## （可选）使用 SDK 操作网络
{: #operate-sdk}

您还可以使用 SDK 来操作区块链网络。本教程说明了如何使用 SDK 将同级加入通道，在同级上安装链代码以及在通道上实例化链代码。这些步骤是可选的，因为如果所有同级都正在 {{site.data.keyword.blockchainfull_notm}} Platform 上运行，那么还可以在 [Swagger UI](/docs/services/blockchain/howto/swagger_apis.html) 中使用“网络监视器”或 API 来执行这些操作。

您需要将 admin signCert 上传到 {{site.data.keyword.blockchainfull_notm}} Platform 来完成这些步骤。您可以在[注册部分](#enroll-app-sdk)末尾找到有关如何上传 signCert 的指示信息。

### 加入通道
{: #join-channel-sdk}

组织使用“网络监视器”或 API 创建或加入通道后，您可以使用 SDK 将同级加入该通道。

1. 通过排序服务[访存通道的起源区块 ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/Channel.html#getGenesisBlock "访存通道的起源区块"){:new_window}。
2. 将起源区块传递到[加入通道 ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/Channel.html#joinChannel "joinChannel"){:new_window} 方法，以将同级加入通道。

要使用 `fabcar` 样本来加入通道，请使用 `invoke.js` 文件作为起始点。您需要以 admin 身份（而非应用程序用户）发送此请求，因此请将 `getUserContext` 方法中的 `user1` 替换为 `admin`。从在 `var request = {` 下定义链代码调用请求的位置开始，根据 [Node SDK 文档 ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/tutorial-channel-create.html "双向 TLS"){:new_window} 中的以下代码片段，将事务流替换为通道加入请求。
  ```
  let g_request = {
    txId :     tx_id
  };

  // 从排序者那里获取起源区块
  channel.getGenesisBlock(g_request).then((block) =>{
    genesis_block = block;
    tx_id = client.newTransactionID();
    let j_request = {
      targets : targets,
      block : genesis_block,
      txId :     tx_id
    };
    // 向同级发送起源区块
    return channel.joinChannel(j_request);
  }).then((results) =>{
    if(results && results[0].response && results[0].response.status == 200) {
    // 良好
    } else {
      // 不佳
    }
  });
  ```

需要先将 signCert 添加到通道，才能访存起源区块。如果在组织加入通道后生成了证书，那么需要将您的 signCert 上传到平台，然后单击“通道”屏幕中的**同步证书**按钮。在发出加入通道的命令之前，您可能需要等待几分钟，以便通道完成同步。有关更多信息，请参阅[管理证书](/docs/services/blockchain/certificates.html)教程中的[将签名证书上传到 {{site.data.keyword.blockchainfull_notm}} Platform](/docs/services/blockchain/certificates.html#upload-certs)。

### 安装链代码
{: #install-cc-sdk}

可以使用 [Fabric 客户机 ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/Client.html "Fabric 客户机"){:new_window} 类中的[安装链代码 ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/Client.html#installChaincode "installChaincode"){:new_window} 方法在同级上安装链代码。

要使用 `fabcar` 样本在同级上安装 `fabcar` 链代码，请使用 `query.js` 文件作为基线并对其进行编辑。您需要以 admin 身份（而非应用程序用户）发送此请求，因此请将 `getUserContext` 方法中的 `user1` 替换为 `admin`。使用以下示例将事务建议对象替换为安装链代码请求：
```
var request = {
    targets: peer,
    chaincodePath: chaincode_path,
    chaincodeId: 'fabcar',
    chaincodeType: 'golang',
    chaincodeVersion: 'v1',
    channelNames: 'mychannel'
	 };
```
{:codeblock}

将此对象发送到 `return fabric_client.installChaincode(request);`，而不是当前该文件中的 `return channel.queryByChaincode(request);` 行。

### 实例化链代码
{: #instantiate-cc-sdk}

要实例化链代码，您需要向同级发送[实例化建议 ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/Channel.html#sendInstantiateProposal "sendInstantiateProposal"){:new_window}，然后向排序服务发送[事务请求 ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/Channel.html#sendTransaction "sendTransaction){:new_window}。

要使用 `fabcar` 样本来实例化链代码，请使用 `invoke.js` 文件作为起始点。您需要以 admin 身份（而非应用程序用户）发送此请求，因此请将 `getUserContext` 方法中的 `user1` 替换为 `admin`。使用以下示例将事务建议对象替换为安装链代码请求：
```
var request = {
    targets: peer,
    chaincodePath: chaincode_path,
    chaincodeId: 'fabcar',
    chaincodeType: 'golang',
    chaincodeVersion: 'v1',
    channelNames: 'mychannel',
    txId : tx_id
};
```
{:codeblock}

将此请求发送到 `return channel.sendInstantiateProposal(request);`，而不是当前该文件中的 `return channel.sendTransactionProposal(request);`。将实例化请求发送到通道后，您需要将已背书建议作为事务发送给排序服务。此操作使用的方法与发送事务的方法相同，因此您可以使文件其余部分保持不变。您可能需要在实例化建议中[增大超时值](#set-timeout-in-sdk)。否则，在平台可以启动链代码容器之前，该请求可能会超时。

需要先将 signCert 添加到通道，才能实例化链代码。如果在您加入通道后生成了证书，那么需要将您的 signCert 上传到平台，然后单击“通道”屏幕中的**同步证书**按钮。在发出实例化链代码的命令之前，您可能需要等待几分钟，以便通道完成同步。要了解更多信息，请参阅[管理证书](/docs/services/blockchain/certificates.html)教程中的[将签名证书上传到 {{site.data.keyword.blockchainfull_notm}} Platform](/docs/services/blockchain/certificates.html#upload-certs)。

## （可选）在 Fabric SDK 中设置超时值
{: #set-timeout-in-sdk}

Fabric SDK 针对区块链网络中的事件在客户机应用程序中设置缺省超时值。请参阅以下有关 Fabric Java SDK 中缺省超时设置的示例。文件路径为 `src\main\java\org\hyperledger\fabric\sdk\helper\Config.java`。

```
    /**
     * Timeout settings
     **/
    public static final String PROPOSAL_WAIT_TIME = "org.hyperledger.fabric.sdk.proposal.wait.time";
    public static final String CHANNEL_CONFIG_WAIT_TIME = "org.hyperledger.fabric.sdk.channelconfig.wait_time";
    public static final String TRANSACTION_CLEANUP_UP_TIMEOUT_WAIT_TIME = "org.hyperledger.fabric.sdk.client.transaction_cleanup_up_timeout_wait_time";
    public static final String ORDERER_RETRY_WAIT_TIME = "org.hyperledger.fabric.sdk.orderer_retry.wait_time";
    public static final String ORDERER_WAIT_TIME = "org.hyperledger.fabric.sdk.orderer.ordererWaitTimeMilliSecs";
    public static final String PEER_EVENT_REGISTRATION_WAIT_TIME = "org.hyperledger.fabric.sdk.peer.eventRegistration.wait_time";
    public static final String PEER_EVENT_RETRY_WAIT_TIME = "org.hyperledger.fabric.sdk.peer.retry_wait_time";
    public static final String EVENTHUB_CONNECTION_WAIT_TIME = "org.hyperledger.fabric.sdk.eventhub_connection.wait_time";
    public static final String EVENTHUB_RECONNECTION_WARNING_RATE = "org.hyperledger.fabric.sdk.eventhub.reconnection_warning_rate";
    public static final String PEER_EVENT_RECONNECTION_WARNING_RATE = "org.hyperledger.fabric.sdk.peer.reconnection_warning_rate";
    public static final String GENESISBLOCK_WAIT_TIME = "org.hyperledger.fabric.sdk.channel.genesisblock_wait_time";

    ...

    // Default values
    /**
     * Timeout settings
     **/
    defaultProperty(PROPOSAL_WAIT_TIME, "20000");
    defaultProperty(CHANNEL_CONFIG_WAIT_TIME, "15000");
    defaultProperty(ORDERER_RETRY_WAIT_TIME, "200");
    defaultProperty(ORDERER_WAIT_TIME, "10000");
    defaultProperty(PEER_EVENT_REGISTRATION_WAIT_TIME, "5000");
    defaultProperty(PEER_EVENT_RETRY_WAIT_TIME, "500");
    defaultProperty(EVENTHUB_CONNECTION_WAIT_TIME, "5000");
    defaultProperty(GENESISBLOCK_WAIT_TIME, "5000");
    /**
     * This will NOT complete any transaction futures time out and must be kept WELL above any expected future timeout
     * for transactions sent to the Orderer. For internal cleanup only.
     */
    defaultProperty(TRANSACTION_CLEANUP_UP_TIMEOUT_WAIT_TIME, "600000"); //10 min.
```
{:codeblock}

但是，您可能需要更改自己应用程序中的缺省超时值。例如，在应用程序调用响应所需时间超过 5000 毫秒（Event Hub 连接的缺省超时值）的事务时，可能会收到失败错误，因为调用事件在 5000 毫秒时结束，而此时事务未完成。您可以设置系统属性以覆盖客户机应用程序的缺省值。因为在设置系统属性前初始化缺省值，所以系统属性可能不会生效。因此，需要在客户机应用程序的静态结构中设置超时系统属性。请参阅以下示例，以了解如何将 Fabric Java SDK 中 Event Hub 连接的超时值更改为 15000 毫秒。文件路径为 `src\main\java\org\hyperledger\fabric\sdk\helper\Config.java`。

```
 public static final String EVENTHUB_CONNECTION_WAIT_TIME = "org.hyperledger.fabric.sdk.eventhub_connection.wait_time";
 private static final long EVENTHUB_CONNECTION_WAIT_TIME_VALUE = 15000;

 static {
     System.setProperty(EVENTHUB_CONNECTION_WAIT_TIME, EVENTHUB_CONNECTION_WAIT_TIME_VALUE);
 }
```
{:codeblock}

如果使用的是 Node SDK，那么可以直接在调用的方法中指定超时值。例如，使用下面的一行将针对[实例化链代码 ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/Channel.html#sendInstantiateProposal "sendInstantiateProposal") 的超时值增加到 5 分钟。
```
channel.sendInstantiateProposal(request, 300000);
```
{:codeblock}

## 使用 CouchDB 时的最佳实践
{: #couchdb-indices}

如果分类帐数据存储在 CouchDB 中，那么强烈建议您为 CouchDB 查询创建索引，并在链代码中使用这些索引。索引允许应用程序在网络添加更多全局状态的事务和条目块时高效地检索数据。另外，CoupchDB 允许您根据通道分类帐上的数据从链代码执行丰富的数据查询。

有关 CouchDB 以及如何设置索引的更多信息，请参阅 Hyperledger Fabric 文档中的 [CouchDB 作为状态数据库 ![外部链接图标](images/external_link.svg "外部链接图标")](http://hyperledger-fabric.readthedocs.io/en/release-1.1/couchdb_as_state_database.html "CouchDB 作为状态数据库"){:new_window}。您还可以在 [Fabric CouchDB 教程 ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/couchdb_tutorial.html) 中找到将索引与链代码配合使用的示例。

避免对将导致扫描整个 CouchDB 数据库的查询使用链代码。全面的数据库扫描将导致很长的响应时间，并且会降低网络的性能。您可以执行以下某些步骤来避免耗时很长的查询：
- 使用链代码设置索引。
- 如果发出丰富的 JSON 查询，请避免将导致全面数据库扫描的运算符，例如 `$or`、`$in` 以及 `$regex`。
- 您应使用 queryLimit 避免返回将导致查询超时的大型数据集。然后，您可以使用多个查询根据限制条件搜索整个数据库。如果使用范围查询，请使用上一个查询返回的最后一个键来启动后续查询。如果要使用丰富的 JSON 查询，请使用数据中的其中一个变量对查询进行排序。然后，使用结果来过滤使用相同变量的下一个查询。
- 不要为了汇总或报告目的而查询整个数据库。如果要构建仪表板或将数据作为应用程序的一部分进行收集，那么可以查询从区块链网络复制数据的链外数据库。这将允许您了解区块链上的数据，而不会降低网络性能或中断事务。

  您可以使用 Fabric SDK 提供的 Channel Event Hub 来构建链外数据存储。例如，您可以使用块侦听器获取添加到通道分类帐的最新事务。然后，可以使用事务读和写集来更新已存储在单独数据库中的全局状态的副本。有关更多信息，请参阅 Node SDK 文档中的
[如何使用基于通道的事件服务 ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/tutorial-channel-events.html "如何使用基于通道的事件服务"){:new_window}。

## 托管应用程序
{: #host-app}

您可以在本地文件系统上托管应用程序，或将其推送到 {{site.data.keyword.Bluemix_notm}}。要将应用程序推送到 {{site.data.keyword.Bluemix_notm}}，请完成以下步骤：
1. 安装 [Cloud Foundry 命令行安装程序 ![外部链接图标](images/external_link.svg "外部链接图标")](https://github.com/cloudfoundry/cli/releases)。使用 `cf` 命令测试安装。
    * 如果安装成功，您应该在终端中看到一堆文本输出。
    * 如果您看到“找不到命令”，那么安装不成功，或者 CF 未添加到系统路径中。
2. 通过发出以下命令来设置 API 端点并使用 {{site.data.keyword.Bluemix_notm}} 标识和密码进行登录：
    ```
    > cf api https://api.ng.bluemix.net
    > cf login
    ```
    {:codeblock}
3. 浏览到应用程序的目录，并通过发出以下命令来推送应用程序。根据应用程序大小，这可能需要几分钟。您可以在终端中查看 {{site.data.keyword.Bluemix_notm}} 中的日志。成功启动应用程序后，日志将停止。
	```
	> cf push YOUR_APP_NAME_HERE
	```
	{:codeblock}
	您可以通过发出以下某个命令来检查应用程序日志：

	* `> cf logs YOUR_APP_NAME_HERE`
	* `> cf logs YOUR_APP_NAME_HERE --recent`

## 断开应用程序与网络的连接
{: #disconnect-app}

完成以下步骤以除去应用程序与 {{site.data.keyword.cloud_notm}} 上的区块链网络之间的连接。
1. 从应用程序配置文件中除去 API 端点信息。有关参考信息，请参阅[将网络 API 端点添加到应用程序](#api-endpoints)。
2. 删除链代码容器。
  1. 在“网络监视器”的“通道”屏幕中，找到已安装链代码的通道。
  2. 在特定通道屏幕中，找到要禁用的链代码。
  3. 单击**删除**按钮，然后单击链代码删除面板中的**提交**。此时将除去链代码容器。
	![删除链代码](images/channel_chaincode.png "删除链代码")
