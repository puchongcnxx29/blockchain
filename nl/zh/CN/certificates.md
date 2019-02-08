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


# 在 {{site.data.keyword.blockchainfull_notm}} Platform 上管理证书
{: #certificates}


***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


{{site.data.keyword.blockchainfull}} Platform 基于 Hyperledger Fabric，可构建许可的区块链网络。参与者被称为网络成员，将持续验证他们的活动和对网络资源的访问。以可信 x509 数字证书形式提供参与者的身份。提供验证的是底层公共密钥基础架构以及用于保护网络中事务和管理的签名/验证操作。
{:shortdesc}

{{site.data.keyword.blockchainfull_notm}} Platform 管理大多数证书操作，而无需用户处理其证书。但是，有时候必须管理证书以允许您与网络通信，例如，开发应用程序或利用远程同级扩展网络。以下是认证中心和底层证书基础架构的简短指南。本教程可协助您了解将使用的证书和需要执行的任务。

## 认证中心
{: #network-ca}

认证中心 (CA) 在网络上提供身份。CA 可被视为充当多个参与方之间的信任锚点的公共可信公证方。给予网络中的所有实体一个根 CA 签署的证书，该证书封装他们的数字身份。此证书是在网络上执行的所有签名和验证操作的信任根。有关如何使用认证中心来建立身份的更多详细信息，请参阅 [Hyperledger Fabric 文档 ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/latest/identity/identity.html)。

网络中的每个成员都拥有自己的 CA。组织 CA 对您拥有的所有实体和组件（例如，管理员、同级或应用程序）的请求添加签名。如果想要向网络添加远程同级或新应用程序，那么需要向认证中心注册新身份（登记）。然后，CA 可向新实体提供该实体与网络交互（注册）所需的证书。

### 使用网络监视器进行注册
{: #ca-panel}

您可以使用 {{site.data.keyword.blockchainfull_notm}} Platform“网络监视器”来查看向 CA 注册的身份以及添加新身份。浏览至“网络监视器”的“认证中心”面板。此面板显示已向 CA 注册的所有身份，包括组织管理员、同级和客户机应用程序。要在组织中注册新身份，请单击面板上的**添加用户**按钮。这样会打开一个弹出窗口，其中显示注册新身份所需的以下字段。
  - **注册标识：**这将是新身份的名称，有时也称为 `enroll ID`。**保存此值**以供在配置远程同级或注册新应用程序时使用。
  - **注册密钥：**这将是您的身份的密码，有时也称为 `enroll Secret`。**保存此值**以供在配置远程同级或注册新应用程序时使用。
  - **类型：**选择想要注册的身份的类型：同级或客户机应用程序。
  - **亲缘关系：**这将是身份所属的组织（如 `org1`）内的亲缘关系。
  - **最大注册数：**您可以使用此字段来限制可使用此身份注册或生成证书的次数。如果保留此字段为空，那么值缺省为不限制注册数。

如果部署[远程同级](/docs/services/blockchain/howto/remote_peer.html)，那么可使用此面板来注册新的同级身份。或者，如果开发可向网络提交事务的应用程序，那么可注册客户机。访问[开发应用程序教程](v10_application.html)以了解有关将 Fabric SDK 用于该平台的更多信息。

### 生成客户机端证书（注册）
{: #enrollment}
您需要进行认证才可以将第三方客户机连接到 {{site.data.keyword.blockchainfull_notm}} Platform。生成必需的证书、专用密钥和公用证书（也称为注册证书或 signCert）的过程被称为注册。客户机与网络通信时，需要这些证书。向网络提交调用的任何客户机都需要使用专用密钥为有效内容签名并附加正确签名的 x509 证书。

访问[开发应用程序教程](v10_application.html)以了解如何[使用光纤网节点 SDK 进行注册](/docs/services/blockchain/v10_application.html#enroll-app)。使用 SDK 进行注册将生成 3 个单独的项：专用密钥、signCert、用于创建 signCert 的公用密钥。

您还可以使用 [Fabric CA 客户机](#enroll-register-caclient)从命令行生成证书。Fabric CA 客户机返回“成员资格服务提供者”(MSP) 文件夹中更完整的证书集合。此文件夹包含 CA 签署的根证书、中间证书、专用密钥以及 signCert。有关 MSP 以及 MSP 文件夹包含的内容的更多信息，请参阅[成员资格服务提供者](#msp)。

仅对已向认证中心注册的身份，您可以使用其名称和密钥生成证书。缺省情况下，**管理员**身份已向 CA 注册，并且列示在“认证中心”屏幕上。可以通过单击“网络监视器”中“概述”屏幕上的**连接概要文件**按钮来找到连接概要文件中管理员身份的密钥。您也可以通过单击“网络监视器”的“认证中心”屏幕上的[添加用户](#ca-panel)按钮来注册新身份，然后使用新身份的名称和密钥生成证书。

**注：**如果遵循指示信息使用上述 Fabric Node SDK 或 Fabric CA 客户机来生成证书，请首先使用管理员身份进行注册。然后使用这些证书来向 CA 注册新客户机身份。如果使用[开发应用程序](/docs/services/blockchain/v10_application.html)中的 SDK 指示信息，那么将使用客户机身份再次注册。然后，您将使用那些证书将事务提交到网络。<!---You can an illustration of how the developing applications tutorial interacts with your organization CA in the diagram below.--->

### 使用网络监视器生成证书
{: #certs-panel}

您可以通过“网络监视器”使用管理员身份生成证书，然后将这些证书直接传递到 SDK。单击您的管理员身份旁边的**生成证书**按钮，以从您的 CA 获取新的 signCert 和专用密钥。**证书**字段包含 signCert，刚好位于**专用密钥**上方。您可以单击每个字段末尾的复制图标以复制值。然后，您需要将这些证书保存在可以将其导入应用程序的位置。有关更多信息，请参阅[开发应用程序教程](/docs/services/blockchain/v10_application.html#enroll-panel)。**请注意**，{{site.data.keyword.blockchainfull_notm}} Platform 不存储这些证书。您需要安全地保存并存储它们。

### 将签名证书上载到 {{site.data.keyword.blockchainfull_notm}} Platform
{: #upload-certs}

应用程序只需要一个有效 signCert 就可以将事务提交到网络。但是，例如，如果客户机想要通过在同级上安装链代码或者将同级加入到通道来运行网络，那么需要将客户机识别为管理员。每个组件识别管理员拥有的一组 signCert。如果您需要从客户机运行网络，那么需要上传 signCert 并将其添加到管理员证书列表。您可以通过在“网络监视器”的[“概述”面板](/docs/services/blockchain/v10_dashboard.html#members)的**证书**选项卡中上传 signCert，在平台上执行此操作。通过点击上传后出现的重新启动按钮，将此证书同步到同级。之后，您的客户机将能够操作网络。您还可以使用 [Swagger API](/docs/services/blockchain/howto/swagger_apis.html) 来上传您的 signCert，以添加管理员证书。

通道还从允许操作通道（包括能够在通道上实例化链代码）的身份中识别一组管理员证书。如果您将新的 signCert 用于远程客户机，您必须将证书同步到通道才可以实例化链代码。在“网络监视器”中执行以下步骤以将证书添加到通道：

1. 浏览至“成员”屏幕的**证书**选项卡。单击**添加证书**按钮将 signCert 上传到平台。
2. 浏览至“通道”屏幕并找到相关通道的名称。
3. 从**操作**标头下的下拉列表中单击**同步证书**，以将新证书添加到通道管理员证书的列表中。

**注：**使用组织的管理员身份生成的证书不会自动视为管理员证书。使用**生成证书**按钮生成的 signCert 和专用密钥将不会允许您的 SDK 操作网络。它们是由 Fabric CA 客户机、SDK 或“网络监视器”生成的，并且与网络组件识别的预先存在的管理员证书列表没有任何连接。您需要在“网络监视器”中上传 signCert，然后 SDK 才能操作网络。

## 使用 TLS 证书
{: #tls}

[传输层安全性 ![外部链接图标](images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_7.1.0/com.ibm.mq.doc/sy10660_.htm) (TLS) 嵌入在 Hyperledger Fabric 的信任模型中。{{site.data.keyword.blockchainfull_notm}} Platform 上的所有组件都使用 TLS 进行认证并相互通信。因此，需要将平台发出的 TLS 证书连接到调用以验证和加密通信。本教程中讨论的其他证书可以为管理网络以及进行网络事务处理提供保护。TLS 证书用于保护对网络的调用。

平台公开发布 TLS 证书，这些证书对于所有网络组件都是相同的。根据成员资格套餐和云位置，您可以使用以下链接下载 TLS 证书。您还可以在[凭证概要文件](/docs/services/blockchain/v10_dashboard.html#connection-profile "凭证概要文件")中查找 TLS 证书。此证书可位于任意位置，只要可从应用程序或命令行进行引用即可。

- 入门套餐的根 TLS 证书
  - 美国：[us01.blockchain.ibm.com.cert ![外部链接图标](images/external_link.svg "外部链接图标")](http://blockchain-certs.mybluemix.net/us01.blockchain.ibm.com.cert "us01.blockchain.ibm.com.cert")；[us02.blockchain.ibm.com.cert ![外部链接图标](images/external_link.svg "外部链接图标")](http://blockchain-certs.mybluemix.net/us02.blockchain.ibm.com.cert "us02.blockchain.ibm.com.cert")
  - 英国：[uk01.blockchain.ibm.com.cert ![外部链接图标](images/external_link.svg "外部链接图标")](http://blockchain-certs.mybluemix.net/uk01.blockchain.ibm.com.cert "uk01.blockchain.ibm.com.cert")；[uk02.blockchain.ibm.com.cert ![外部链接图标](images/external_link.svg "外部链接图标")](http://blockchain-certs.mybluemix.net/uk02.blockchain.ibm.com.cert "uk02.blockchain.ibm.com.cert")
  - 悉尼：[aus01.blockchain.ibm.com.cert ![外部链接图标](images/external_link.svg "外部链接图标")](http://blockchain-certs.mybluemix.net/aus01.blockchain.ibm.com.cert "aus01.blockchain.ibm.com.cert")<!--；[aus02.blockchain.ibm.com.cert ![外部链接图标](images/external_link.svg "外部链接图标")](http://blockchain-certs.mybluemix.net/aus02.blockchain.ibm.com.cert "aus02.blockchain.ibm.com.cert")-->
- [企业套餐的根 TLS 证书 ![外部链接图标](images/external_link.svg "外部链接图标")](https://blockchain-certs.mybluemix.net/3.secure.blockchain.ibm.com.rootcert)

所有 {{site.data.keyword.blockchainfull_notm}} Platform 网络都使用服务器端 TLS，网络需要在其中认证客户机。企业套餐网络还可支持相互 TLS，其中客户机和服务器相互认证以进一步保护您的应用程序。客户机端 TLS 证书（用于相互 TLS）由客户机 CA 颁发并且对于网络是唯一的。如果使用企业套餐网络，那么建议启用相互 TLS。有关相互 TLS 的更多信息，请参阅这些[指示信息](/docs/services/blockchain/v10_dashboard.html#mutual-tls "相互 TLS 指示信息")。

### 从 TLS 证书中检索域名

当您与位于 {{site.data.keyword.blockchainfull_notm}} Platform 外部的 Hyperledger Fabric 组件进行通信时，必须使用正确的域名来提交您的调用。如果在未解析为该组件域名的情况下向组件的 IP 地址提交了调用，那么该调用将被拒绝并返回错误。

您可以在连接概要文件中找到平台上托管的组件的完整 URL 和域名。以下示例显示同级的完整 URL，该 URL 与同级的域名 `us01.blockchain.ibm.com` 并置在一起。
```
grpcs://n7413e3b503174a58b112d30f3af55016-org1-peer1.us01.blockchain.ibm.com:31002"
```

您还可以从其 TLS 证书中找到组件的域名。

1. 从上面的列表中下载其中一个根 TLS 证书，并将其保存在本地机器上。
2. 在 root 用户 TLS 证书所在的同一目录中，运行以下命令。此命令可以在命令行中以人类可读的格式显示证书。然后，可以在命令行中找到重要信息，例如域名。
  ```
  openssl x509 -in <certificate file> -text
  ```
  {:codeblock}

例如，如果您下载所列示的第一个证书，并发出命令 `openssl x509 -in us01.blockchain.ibm.com.cert -text`，那么在输出中您可以看到以下代码。

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            05:5c:42:fb:90:b9:cb:3d:60:7c:e4:ec:7f:7f:8e:38
    Signature Algorithm: ecdsa-with-SHA256
        Issuer: C=US, O=DigiCert Inc, CN=DigiCert ECC Secure Server CA
        Validity
            Not Before: Jun 11 00:00:00 2018 GMT
            Not After : Jun 19 12:00:00 2019 GMT
        Subject: C=US, ST=New York, L=Armonk, O=International Business Machines Corporation, CN=*.us01.blockchain.ibm.com
    ...
    ...
```

域名在主题行中列示为 `CN=*.us01.blockchain.ibm.com`。您还可以找到在证书中更靠下的位置列出的备用域名。

如果您与远程同级或在 {{site.data.keyword.blockchainfull_notm}} Platform 外部托管的 Fabric 组件进行通信，那么从 TLS 证书检索组件的域名可能会很有用。然后，可以在使用 SDK 时将域名添加到 SSL 覆盖，或者将域名和相应的 IP 地址添加到 `etc.hosts` 文件中。

## 成员资格服务提供者 (MSP)
{: #msp}

{{site.data.keyword.blockchainfull_notm}} Platform 的组件通过成员资格服务提供者 (MSP) 使用身份。MSP 将 CA 颁发的证书与网络和通道角色相关联。有关 MSP 的更多信息，请参阅 [Hyperledger Fabric 文档中的成员资格概念主题 ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/latest/membership/membership.html "Hyperledger Fabric 文档中的成员资格概念主题")。

Fabric 中的 MSP 文件夹具有已定义的结构。在使用 Fabric CA 客户机进行注册时，客户机将证书存储在本地文件系统上 MSP 文件夹的以下子文件夹中：

- **cacerts：**此文件夹包含网络的根 CA 的根证书。
- **intermediatecerts：**这些是网络的中间 CA 的证书。这些中间 CA 链接到根 CA 并构成信任链。每个企业套餐组织具有两个中间 CA 以实现故障转移和高可用性。
- **signcerts：**此文件夹包含公共签名证书，也称为 signCert 或注册证书。在从命令行引用 MSP 目录或者使用 SDK 构建用户上下文对象时，此证书连接到网络调用（例如，链代码调用）。如果想要通过 SDK 或命令行运行网络，那么可将此证书上传到平台。
- **keystore：**此文件夹包含专用密钥。在从命令行引用 MSP 目录或者使用 SDK 构建用户上下文对象时，使用此密钥来对网络调用签名。请妥善保管此密钥，以保护网络和数据安全。

您还可以使用“网络监视器”和 Swagger API 构建 Fabric CA 客户机可引用的 MSP 文件夹。

- **cacerts** 和 **intermediatecerts**：您可以通过向 MSP API 发出 Get 请求，使用 [Swagger API](/docs/services/blockchain/howto/swagger_apis.html) 访存这些证书。
- **signcerts** 和 **keystore**：您可以通过单击“认证中心”面板上的**生成证书**按钮，生成这些证书。这样将打开一个列出两个证书的弹出窗口。复制并存储 signcerts 中的**证书**和 keystore 中的**专用密钥**。请将这些证书保存在安全位置，因为这些证书未存储在平台上。

多个 Fabric 组件在其 MSP 文件夹中包含其他信息。例如，如果运行远程同级，那么可能会看到以下文件夹：

- **admincerts：**此文件夹包含此组织或组件的管理员列表。如果从命令行或 SDK 操作远程同级，那么需要将 signCert 上传到此文件夹。如果使用 Fabric CA 客户机，那么在 MSP 中还需要一个 admincerts 文件夹，该文件夹包含要被识别为管理员证书的对应 signCert。
- **tls：**此文件夹存储用于与其他网络组件通信的 TLS 证书。

有关 MSP 结构的更多信息，请参阅 Hyperledger Fabric 文档中的[成员资格 ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/latest/membership/membership.html "成员资格") 和[成员资格服务提供者 ![外部链接图标](images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/latest/msp.html "成员资格服务提供者")。

## 使用 Fabric CA 客户机注册和登记
{: #enroll-register-caclient}

您还可以使用 Fabric CA 客户机来生成证书并向认证中心注册新身份。以下指示信息使用您的管理员身份生成证书，然后使用这些证书来注册新客户机。有关使用 Fabric CA 客户机进行注册和生成证书的更多信息，请参阅[成员资格服务提供程序](#msp)。

### 使用 Fabric CA 客户机进行注册
{: #enroll-app-caclient}

1. 将 [fabric-ca 二进制文件夹](https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric-ca/hyperledger-fabric-ca/)下载到本地计算机并解压缩，然后将它们移至文件夹中，例如，`$HOME/fabric-ca-platform/`。切换到将客户机二进制文件移至的目录，从而可直接在命令中进行引用。
    ```
    cd $HOME/fabric-ca-platform/
    ```
    {:codeblock}

2.  设置客户机可在其中创建密钥的路径。请确保除去过去尝试创建的配置资料。如果之前未运行 `enroll` 命令，那么 `msp` 文件夹和 `.yaml` 文件不存在。
    ```
    export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca-platform
    rm -rf $FABRIC_CA_CLIENT_HOME/fabric-ca-client-config.yaml
    rm -rf $FABRIC_CA_CLIENT_HOME/msp
    ```
    {:codeblock}

3. 根据使用的服务套餐、位置和集群，从 {{site.data.keyword.cloud_notm}} 下载 TLS 证书。您可以根据认证中心的 URL 来查找服务套餐。
  - 入门套餐的根 TLS 证书
    - 美国：[us01.blockchain.ibm.com.cert ![外部链接图标](images/external_link.svg "外部链接图标")](http://blockchain-certs.mybluemix.net/us01.blockchain.ibm.com.cert "us01.blockchain.ibm.com.cert")；[us02.blockchain.ibm.com.cert ![外部链接图标](images/external_link.svg "外部链接图标")](http://blockchain-certs.mybluemix.net/us02.blockchain.ibm.com.cert "us02.blockchain.ibm.com.cert")
    - 英国：[uk01.blockchain.ibm.com.cert ![外部链接图标](images/external_link.svg "外部链接图标")](http://blockchain-certs.mybluemix.net/uk01.blockchain.ibm.com.cert "uk01.blockchain.ibm.com.cert")；[uk02.blockchain.ibm.com.cert ![外部链接图标](images/external_link.svg "外部链接图标")](http://blockchain-certs.mybluemix.net/uk02.blockchain.ibm.com.cert "uk02.blockchain.ibm.com.cert")
    - 悉尼：[aus01.blockchain.ibm.com.cert ![外部链接图标](images/external_link.svg "外部链接图标")](http://blockchain-certs.mybluemix.net/aus01.blockchain.ibm.com.cert "aus01.blockchain.ibm.com.cert")<!--；[aus02.blockchain.ibm.com.cert ![外部链接图标](images/external_link.svg "外部链接图标")](http://blockchain-certs.mybluemix.net/aus02.blockchain.ibm.com.cert "aus02.blockchain.ibm.com.cert")-->
  - [企业套餐的根 TLS 证书 ![外部链接图标](images/external_link.svg "外部链接图标")](https://blockchain-certs.mybluemix.net/3.secure.blockchain.ibm.com.rootcert)

  将内容保存到文件夹，例如，``$HOME/tls``。此步骤允许在连线上加密数据流。

4. 从“网络监视器”中的“概述”面板打开**连接概要文件** JSON 文件，并找到以下变量：
  * CA 的 URL：``certificateAuthorities`` 下的 ``url``
  * 管理用户标识：``enrollId``
  * 管理密码：``enrollSecret``
  * CA 名称：``caName``

5. 您可以使用 Fabric CA 客户机，通过使用以下命令传入 TLS 证书路径和上述 4 个字符串，向认证中心发送 `enroll` 调用：
  ```
  ./fabric-ca-client enroll -u https://<enroll_id>:<enroll_password>@<ca_url_with_port> --caname <ca_name> --tls.certfiles <tls_cert_path>
  ```
  {:codeblock}

  您需要在将 Fabric CA 客户机移至的目录中运行此命令。实际调用可能类似于以下示例命令：
  ```
  ./fabric-ca-client enroll -u https://admin:dda0c53f7b@n7413e3b503174a58b112d30f3af55016-org1-ca.us3.blockchain.ibm.com:31011 --caname org1CA --tls.certfiles $HOME/tls/us2.blockchain.ibm.com.cert
  ```

6. 在 `$FABRIC_CA_CLIENT_HOME/msp/signcerts/cert.pem` 中查找您的管理证书。然后，您可以从“网络监视器”将管理证书上传到区块链网络。有关添加证书的更多信息，请参阅“网络监视器”中[“成员”面板的“证书”选项卡](/docs/services/blockchain/v10_dashboard.html#members)。

  您还可以在以下目录中找到 CA 根证书和管理专用密钥：
  * CA 根证书：`$FABRIC_CA_CLIENT_HOME/msp/cacerts/--<ca_name>.pem`
  * 管理专用密钥：`$FABRIC_CA_CLIENT_HOME/msp/keystore/<>_sk file`

有关使用 Fabric CA 客户机进行注册和使用生成的证书来操作网络组件的示例，请参阅[操作远程同级](/docs/services/blockchain/howto/peer_operate_icp.html#peer-cli-operate)的指示信息。

### 使用 Fabric CA 客户机进行登记
{: #register-app-caclient}

1. 发出以下命令以查找区块链网络中的亲缘关系和组织名称。
  ```
  ./fabric-ca-client affiliation list --tls.certfiles <tls_cert_path>
  ```
  {:codeblock}

  您应该看到与以下示例类似的信息：
  ```
  affiliation: ibp
      affiliation: ibp.PeerOrg1
  ```

  记下第二个 **affiliation** 值，例如 `ibp.PeerOrg1`。您需要在下面的命令中使用此值。

2. 运行以下命令以注册同级。
  ```
  ./fabric-ca-client register --id.name <name> --id.affiliation <affiliation> --id.secret <password> --tls.certfiles <tls_cert_path>
  ```
  {:codeblock}

  指定同级的名称和密码，并将 `name` 和 `password` 替换为同级名称和密码。请记下这些信息。配置同级时，需要此信息。例如：
  ```
  ./fabric-ca-client register --id.name user1 --id.affiliation ibp.PeerOrg1 --id.secret userpw  --tls.certfiles $HOME/tls/us2.blockchain.ibm.com.cert
  ```

  您只能注册一次身份。如果遇到问题，请尝试使用新的用户名和密码来执行命令。

  命令成功完成后，应该会看到类似于以下示例的信息：
  ```
  2018/06/18 16:53:00 [INFO] Configuration file location: /fabric-ca-platform/admin/fabric-ca-client-config.yaml
  2018/06/18 16:53:00 [INFO] TLS Enabled
  2018/06/18 16:53:00 [INFO] TLS Enabled
  Password: userpw
  ```
