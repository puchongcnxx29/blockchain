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

# 在使用入门套餐或企业套餐的 {{site.data.keyword.cloud_notm}} Private 上操作同级
{: #peer-operate_icp}

***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

在 {{site.data.keyword.cloud_notm}} Private (ICP) 上设置 {{site.data.keyword.blockchainfull}} Platform 同级后，您需要完成若干操作步骤，同级才能向入门套餐或企业套餐网络提交交易。这些步骤包括将组织添加到通道，将同级连接到通道，在同级上安装链代码，在通道上实例化链代码，以及将应用程序连接到同级。
{:shortdesc}

您需要从 ICP 集群完成一些必备步骤才能操作同级。

**先决条件：**
- [配置 CLI](#peer-kubectl-configure)
- [检索同级端点信息](#peer-endpoint)
- [下载同级 TLS 证书](#peer-tls)

然后，可以使用下列其中一种方法来操作同级。

**操作路径：**
- [使用 Fabric SDK](#peer-operate-with-sdk)
- [使用命令行](#peer-cli-operate)

Fabric SDK 是建议的路径，不过指示信息假定您熟悉 SDK 的操作。如果首选使用命令行，那么可以使用 Fabric 同级客户机。

<!--
It is recommended that you deploy at least two instances of the peer Helm chart for [high availability](/docs/services/blockchain/peer_icp.html#high-availability). Therefore, you need to follow these operations steps once for each peer. When you are ready to invoke and query chaincode from your application, connect to both peers to ensure that your [applications are highly available](/docs/services/blockchain/v10_application.html#ha-app).
-->

**注**：{{site.data.keyword.blockchainfull_notm}} Platform 同级无权访问在 {{site.data.keyword.blockchainfull_notm}} Platform 上托管的同级的完整功能，也无权获得完全支持。因此，无法使用“网络监视器”运行 ICP 上的同级。开始运行同级之前，请确保查看[注意事项和限制](/docs/services/blockchain/ibp-for-icp-about.html#ibp-icp-considerations)。

## 先决条件

无论您是计划使用 SDK 还是命令行，都需要在操作同级的过程中完成此部分中的步骤。您可以在开始之前完成所有这些步骤。

### 配置 CLI
{: #peer-kubectl-configure}

您需要使用 **kubectl** 命令行工具来连接到在 ICP 中运行的同级容器。

1. 登录到 ICP 集群 UI。浏览至**命令行工具**选项卡，然后单击 **Cloud Private CLI**。您将看到以下可以下载的工具。

   * 安装 IBM Cloud Private CLI 和插件
   * 安装 Kubectl CLI
   * 安装 Helm
   * 安装 Istio CLI

  要操作同级，您需要使用前**三个**工具，其中 Helm 是可选的。单击每个工具，然后针对您使用的机器类型，运行相应的 `curl` 命令。接下来，对每个工具发出 `chmod` 和 `sudo mv` 命令。`chmod` 命令将更改相关 CLI 的许可权以使其可执行，`sudo mv` 命令将移动文件并对其重命名。

  第一个工具 **cloudctl** 的命令可能类似于以下示例：

  ```
  chmod +x cloudctl-darwin-amd64<suffix of your binary>
  sudo mv path/to/cloudctl-darwin-amd64<suffix> /usr/local/bin/cloudctl
  ```
  {:codeblock}

  第二个工具 **kubectl** 的命令可能类似于以下示例：

  ```
  chmod +x kubectl-darwin-amd64<suffix of your binary>
  sudo mv path/to/kubectl-darwin-amd64<suffix> /usr/local/bin/kubectl
  ```
  {:codeblock}

2. 配置 **kubectl** 后，运行以下命令：

  ```
  kubectl get pods
  ```
  {:codeblock}

  如果该命令成功，将返回类似于以下示例的响应，其中 `peer-6cf85f6687-hcxpl` 是同级容器的名称。

  ```
  NAME                                 READY      STATUS             RESTARTS   AGE
  peer-6cf85f6687-hcxpl          2/2        Running            0          6h
  ```

  您现在已准备就绪，可以使用 **kubectl** 工具来获取同级端点信息。

3. （可选）如果要使用 **Helm**，请再完成若干步骤。请注意，Helm 是可选的，因此在这些指示信息中您不需要使用此工具。但是，对于管理 Helm 发布并创建您自己的归档以部署在 ICP 中，此工具非常有用。

  1. 在 ICP UI 中，单击“安装 Helm”并运行 `curl` 命令。
  2. 通过运行以下命令来解压 `tar` 文件：

    ```
    tar -xzvf helm-darwin-amd64<suffix>
    ```
    {:codeblock}

  3. 运行 `sudo mv` 命令，并将 `/helm` 附加到 `darwin-amd64`。请注意，无需对 Helm 运行 `chmod` 命令。例如：

    ```
    sudo mv darwin-amd64/helm/ /usr/local/bin/helm
    ```
    {:codeblock}

  您可以运行 `helm help` 命令来确认是否成功安装了 Helm。

### 检索同级端点信息
{: #peer-endpoint}

您需要通过 SDK 或 Fabric CA 客户机将同级端点设定为目标，才能加入通道或安装智能合同。可以使用 ICP 控制台 UI 来查找同级的端点。您需要是[集群管理员 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/user_management/assign_role.html "集群管理员角色和操作") 才能完成以下步骤：

1. 登录到 ICP 控制台，然后单击左上角的**菜单**图标。
2. 单击**工作负载** > **Helm 发布**。
3. 找到您的 Helm 发布的名称，然后打开 Helm 发布详细信息面板。
4. 向下滚动到面板底部的**注释**部分。在**注释**部分中，您会看到一组命令，可帮助您操作同级部署。
5. 如果尚未配置 [kubectl CLI](#peer-kubectl-configure)，请执行相应步骤进行配置。您将使用 kubectl 来与同级容器进行交互。
6. 在 CLI 中，运行注释中的第一个命令，此命令位于 **1. 通过运行以下命令获取应用程序 URL：**之后。此命令将显示同级 URL 和端口，类似于以下示例。保存此 URL 以供未来命令使用。

  ```
  http://9.30.94.174:30159
  ```
  {:codeblock}

**注：**如果将同级部署在防火墙后，那么需要打开传递，以支持平台上的网络与同级完成 TlS 握手。您只需要对绑定到同级端口 7051 的 Node 端口启用传递。有关更多信息，请参阅[查找同级端点信息](#peer-endpoint)。

### 下载同级 TLS 证书
{: #peer-tls}

您需要下载同级 TLS 证书，并将其传递到命令，才能通过远程客户机与同级进行通信。

1. 登录到 ICP 控制台，然后单击左上角的**菜单**图标。
2. 单击**工作负载** > **Helm 发布**。
3. 找到您的 Helm 发布的名称，然后打开 Helm 发布详细信息面板。
4. 向下滚动到面板底部的**注释**部分。在**注释**部分中，您会看到一组命令，可帮助您操作同级部署。
5. 如果尚未配置 [kubectl CLI](#peer-kubectl-configure)，请执行相应步骤进行配置。您将使用 kubectl 来与同级容器进行交互。
6. 在 CLI 中，运行注释中的第一个命令，此命令位于 **3. 获取同级 TLS 根证书文件**之后。此命令会将 TLS 证书作为 `cacert.pem` 文件保存在本地计算机上。
7. 将该证书移至可在未来的命令中引用该证书的位置，然后将其重命名为 `peertls.pem`。

  ```
  mkdir $HOME/fabric-ca-client/peer-tls
  cp cacert.pem $HOME/fabric-ca-client/peer-tls/peertls.pem
  ```
  {:codeblock}

## 使用 Fabric SDK 来操作同级
{: #peer-operate-with-sdk}

Hyperledger Fabric SDK 提供了一组功能强大的 API，支持应用程序与区块链网络进行交互并操作区块链网络。您可以在 [Hyperledger Fabric SDK 社区文档 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/getting_started.html#hyperledger-fabric-sdks "Hyperledger Fabric SDK 社区文档") 中找到 Fabric SDK 中最新的受支持语言列表和完整的可用 API 列表。可以使用 Fabric SDK 将同级加入 {{site.data.keyword.blockchainfull_notm}} Platform 上的通道，在同级上安装链代码，然后在通道上实例化链代码。

以下指示信息使用 [Fabric Node SDK ![外部链接图标](../images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/ "Fabric Node SDK") 来操作同级，并假定您先前已经熟悉该 SDK。开始之前，可以使用[开发应用程序教程](../v10_application.html)来学习如何使用 Node SDK，如果已准备好调用和查询链代码，可以将此教程作为使用同级开发应用程序的指南。

### 安装 Node SDK

可以使用 NPM 来安装 [Node SDK ![外部链接图标](../images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/ "Node SDK")：

```
npm install fabric-client@1.2
```
{:codeblock}

建议您使用 Node SDK V1.2。

### 准备 SDK 以使用同级
{: #remote-peer-node-sdk}

同级将使用内部同级管理员的 signCert 进行部署。这允许您使用同级管理员的证书和 MSP 文件夹来操作同级。

找到在[注册同级管理员](/docs/services/blockchain/peer_deploy_ibp.html#enroll-admin)时创建的证书。如果使用的是示例命令，那么可以在 `$HOME/fabric-ca-client/peer-admin` 中找到同级管理员的 MSP 文件夹。
  - 可以使用 MSP 文件夹中的 signCert（公用密钥）和专用密钥通过 SDK 来构建同级管理员用户上下文。您可以在以下位置找到这些键：
    - signCert 可以在 **signcerts** 文件夹 `$HOME/fabric-ca-client/peer-admin/msp/signcerts` 中找到
    - 专用密钥可以在 **keystore:** 文件夹 `$HOME/fabric-ca-client/peer-admin/msp/keystore` 中找到
    您可以在[开发应用程序教程的此部分](/docs/services/blockchain/v10_application.html#enroll-panel)中找到如何仅使用公用和专用密钥来构造用户上下文并运行 SDK 的示例。

您还可以使用 SDK 通过入门套餐或企业套餐上 CA 的端点信息以及[同级管理员用户名和密码](/docs/services/blockchain/peer_deploy_ibp.html#register-admin)，生成同级管理员的 signCert 和专用密钥。

### 将同级管理员的 signCert 上传到入门套餐或企业套餐
{: #remote-peer-upload-sdk}

您需要将同级管理员的 signCert 上传到 {{site.data.keyword.blockchainfull_notm}} Platform 上的网络，以便应用程序有权操作网络。

- 在 SDK 生成的加密资料所在的目录下以同级管理员命名的文件中，可以找到 signCert。复制 `certificate` 字段后引号内的证书，证书以 `-----BEGIN CERTIFICATE-----` 开头，并以 `-----END CERTIFICATE-----` 结尾。可以使用 CLI 通过发出 `echo -e"<CERT>" > admin.pem` 命令，将证书转换为 PEM 格式。然后，可以使用“网络监视器”将证书的内容粘贴到区块链网络中。登录到 {{site.data.keyword.blockchainfull_notm}} Platform 上的网络。在“网络监视器”的“成员”屏幕上，单击**证书** > **添加证书**。为证书指定任意名称，然后将内容粘贴到**证书**字段中。当系统询问您是否要重新启动同级时，请单击**重新启动**。必须在组织加入通道之前执行此操作。

### 将同级的 TLS 证书传递到 SDK
{: #icp-peer-download-tlscert}

您需要引用同级的 TLS 证书来认证与 SDK 的通信。找到[从同级容器下载](#peer-tls)的 TLS 证书，并将其保存在应用程序可以对其进行引用的位置。如果使用的是示例命令，那么可以在 `$HOME/fabric-ca-client/peer-tls/peertls.pem` 中找到同级 TLS 证书。然后，可以使用简单的读取文件命令将该 TLS 证书导入应用程序。

```
var peerTLSCert = fs.readFileSync(path.join(__dirname, './peertls.pem'));
```
{:codeblock}

### 向 SDK 提供同级端点信息
{: #peer-SDK-endpoints}

找到[同级的端点信息](#peer-endpoint)，然后通过声明新的同级变量或更新连接概要文件，向 SDK 提供这些信息。以下示例将同级定义为 Fabric 网络上的端点，并向其传递导入的 TLS 证书。

```
var peer = fabric_client.newPeer('grpcs://9.30.94.174:30167', { pem:  Buffer.from(peerTLSCert).toString(), 'ssl-target-name-override': null});
```
{:codeblock}

<!--
You need to specify a `ssl-target-name-override` of `<something>.blockchain.com` in order for the peer to resolve the DNS request.
-->

**注：**由于同级位于 {{site.data.keyword.cloud_notm}} 外部，因此 {{site.data.keyword.blockchainfull_notm}} Platform 网络上的其他组织将无法在其连接概要文件中找到同级的端点信息。如果其他组织需要向同级发送交易以进行背书，您需要在频带内和频带外操作中向这些组织提供同级 URL。

### 使用 SDK 加入通道
{: #peer-join-channel-sdk}

您的组织需要成为通道的成员，然后才能将同级加入通道。

  - 可以为同级启动新的通道。如果您是通道启动者，那么可以在[通道创建](/docs/services/blockchain/create_channel.html#creating-a-channel)期间自动包含您的组织。

  - 区块链网络的其他成员也可以使用[通道更新](/docs/services/blockchain/create_channel.html#updating-a-channel)将您的组织添加到现有通道。

    将组织添加到通道后，需要将同级的 signCert 添加到该通道，以便其他成员可以在交易期间验证您的数字签名。在安装期间，同级会上传其 signCert，这意味着您只需要将该证书同步到通道即可。在“网络监视器”的“通道”屏幕中，找到组织加入的通道，然后从**操作**标题下的下拉列表中选择**同步证书**。此操作会同步通道上所有同级的证书。您可能需要等待几分钟，以便完成通道同步后，再发出 `join channel` 命令。

    **注：**仅当要添加到 ICP 并连接到入门套餐或企业套餐网络的同级与已使用“网络监视器”添加的同级属于同一组织时，才能在“网络监视器”的“通道”屏幕中查看正在添加到通道的新区块、正在实例化的链代码以及其他通道更新。这是因为“网络监视器”是在“通道”屏幕上收集同级的信息，因此无法查看 {{site.data.keyword.cloud_notm}} 外部的同级。如果没有任何同级在使用“专用数据”功能，那么“网络监视器”中针对组织中的不同同级的信息相同。

在组织成为通道的成员后，请遵循指示信息使用 SDK [将同级加入通道](/docs/services/blockchain/v10_application.html#join-channel-sdk)。您需要提供排序服务的 URL 和通道名称。

### 使用 SDK 在同级上安装链代码
{: #peer-install-cc-sdk}

使用以下指示信息通过 SDK 在同级上[安装链代码](/docs/services/blockchain/v10_application.html#install-cc-sdk)。

### 使用 SDK 在通道上实例化链代码
{: #peer-instantiate-cc-sdk}

只需要通道中的一个成员对链代码进行实例化或更新。因此，{{site.data.keyword.blockchainfull_notm}} Platform 上同级的通道的任何网络成员都可以使用“网络监视器”来实例化链代码并指定背书策略。但是，如果要使用同级在通道上实例化链代码，那么可以使用 SDK 并遵循指示信息来[实例化链代码](/docs/services/blockchain/v10_application.html#instantiate-cc-sdk)。

## 使用 CLI 来操作同级
{: #peer-cli-operate}

您还可以通过命令行使用 Fabric `同级`客户机来操作同级。

同级已使用内部同级管理员的 signCert 进行部署，允许该身份操作同级。以下指示信息将使用在[部署同级](/docs/services/blockchain/peer_deploy_ibp.html#register-admin)时生成的同级管理员的 MSP 文件夹来将同级加入通道，在同级上安装链代码，然后在通道上实例化链代码。

### 下载 Fabric 同级客户机
{: #peer-client}

获取同级客户机的最简单方法是从 Hyperledger 下载所有 Fabric 工具二进制文件。浏览至要在其中使用命令行下载二进制文件的目录，然后可通过发出以下命令来访存这些二进制文件。如果尚未安装 [Curl ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/prereqs.html#install-curl "Curl")，那么需要首先安装 Curl。

```
curl -sSL http://bit.ly/2ysbOFE | bash -s 1.2.1 1.2.1 -d -s
```
{:codeblock}

该命令将在 `bin/` 目录中安装二进制文件。设置工具的路径，即上述 Fabric 工具的下载位置：
```
export PATH=$PATH:<full_path_to_bin_folder>
```
{:codeblock}

例如，如果已将二进制文件安装在主目录中，那么应将 `PATH` 设置为：

```
export PATH=$PATH:$HOME/bin
```
{:codeblock}

下载二进制文件将创建包含 core.yaml、order.yaml 和 configtx.yaml 文件的 `config` 文件夹。将 Fabric 配置路径设置为此 `config` 目录：

```
export FABRIC_CFG_PATH=<full_path_to_config_folder>
```
{:codeblock}

例如：
```
export FABRIC_CFG_PATH=$HOME/config
```
{:codeblock}


### 管理本地系统上的证书
{: #manage-certs}

切换到在其中生成同级管理员的 MSP 文件夹的目录。如果您执行的是本文档中的示例步骤，那么可以在类似以下目录的目录中找到 MSP 文件夹：

```
cd $HOME/fabric-ca-client/peer-admin/msp
```
{:codeblock}

需要在本地计算机上对证书执行一些管理工作，才能操作同级。例如，您需要将同级管理员的 signCert 上传到入门套餐或企业套餐，并确保您可以从 {{site.data.keyword.cloud_notm}} 上的同级和网络访问 TLS 证书。有关要使用的证书和要执行的任务的更多信息，请参阅[在 {{site.data.keyword.blockchainfull_notm}} Platform 上管理证书](/docs/services/blockchain/certificates.html)。

1. 将同级管理员的 signCert 移至名为 `admincerts` 的新文件夹：

  ```
  cd $HOME/fabric-ca-client/peer-admin/msp
  mkdir admincerts
  cp signcerts/cert.pem admincerts/cert.pem
  ```
  {:codeblock}

2. 将此 signCert 上传到 {{site.data.keyword.cloud_notm}} 上的网络，以便远程 CLI 或应用程序可以执行通道操作，例如访存通道起源区块或实例化链代码。
    1. 在本地计算机上，打开 `HOME/fabric-ca-client/peer-admin/msp/admincerts/cert.pem` 文件，然后将其复制到剪贴板。
    2. 进入 {{site.data.keyword.blockchainfull_notm}} Platform 上网络的“网络监视器”。
    3. 单击左侧导航器中的**成员**，然后单击**证书**选项卡。
    4. 单击**添加证书**按钮。
    5. 在**添加证书**窗口中，为证书提供名称（如“peer-admin”），粘贴刚才复制到剪贴板的证书，然后单击**提交**。
    6. 系统将询问您是否要重新启动同级。单击**重新启动**。
    7. 单击左侧导航器中的**通道**，并找到同级将加入的通道。
    8. 单击**操作**标题下的三个点，然后单击**同步证书**。在**同步证书**窗口中，单击**提交**。

    **注**：在同级加入通道或在通道上实例化链代码之前，务必同步通道证书。您可能需要等待几分钟，以便完成通道同步后，再发出加入通道或实例化链代码的命令。

3. 确保您已[下载同级 TLS 证书](/docs/services/blockchain/#peer-tls)，并且可以通过命令行引用该证书。如果遵循的是示例命令，那么可以在 `$HOME/fabric-ca-client/peer-tls/peertls.pem` 文件中找到此 TLS 证书。

4. 在[注册同级管理员](/docs/services/blockchain/peer_deploy_ibp.html#enroll-admin)时，还需要引用用于与入门套餐或企业套餐 CA 进行通信的 TLS 证书。如果遵循的是本文档中的示例命令，那么可以在 `$HOME/fabric-ca-client/tls-ibp/tls.pem` 文件中找到此 TLS 证书。

您可以运行 tree 命令来验证是否已完成这些步骤。浏览至存储证书的目录。tree 命令应该生成类似于以下结构的结果：
```
cd $HOME/fabric-ca-client
tree
.
├── ca-admin
│   ├── fabric-ca-client-config.yaml
│   └── msp
│       ├── cacerts
│       │   └── 9-12-19-115-31873-SampleOrgCA.pem
│       ├── keystore
│       │   └── c44ec1e708f84b6d0359f58ce2c9c8a289919ba81f2cf4bb5187c4ad5a43cbb0_sk
│       └── signcerts
│       |   └── cert.pem
│       └── user
├── catls
│   └── tls.pem
├── peer-admin
│   ├── fabric-ca-client-config.yaml
│   └── msp
│       ├── admincerts
│       │   └── cert.pem
│       ├── cacerts
│       │   └── n4790a319014a473a8a65850c660396ab-org1-ca-us05-blockchain-ibm-com-31011-org1CA.pem
│       ├── keystore
│       │   └── 6c2a999ee80a6b0592c63c58f95193bea2df59efe5489938d513de47b83b372a_sk
│       ├── signcerts
│       │   └── cert.pem
│       └── user
├── peer-tls
│   └── peertls.pem
├── tls-ibp
│   └── tls.pem
├── tls.pem
└── tlsca-admin
    ├── fabric-ca-client-config.yaml
    └── msp
        ├── cacerts
        │   └── 9-30-250-70-30395-tlsca.pem
        ├── keystore
        │   └── bd57fa20283dfc76ada83f989ee0f62ce23e98c94dbd26f6cd23202d8084e38e_sk
        ├── signcerts
        │   └── cert.pem
        └── user
```

### 设置 CLI 环境变量
{: #environment-variables}

将所有证书都移至必需的位置后，您需要设置一些环境变量供命令使用。然后，即准备就绪可以使用 Fabric 同级客户机来操作同级。

1. 在“网络监视器”的**概述**屏幕中可用的**连接概要文件**文件中检索入门套餐或企业套餐中的配置信息。单击**连接概要文件**，然后单击**下载**。

  -  通过搜索`排序者 > URL` 下的**排序者**来查找排序者 URL。记下以网络名称开头的 URL。该 URL 类似于以下示例：
    

    ```
    ash-zbc07b.4.secure.blockchain.ibm.com:21239
    ```

  - 通过搜索**组织**来查找组织的名称。这应该是您用于注册同级的组织。您可以找到组织的名称及其关联的 `mspid`。请记下 `mspid` 的值。

2. [查找同级的端点信息](#peer-endpoint)。您需要使用同级端点来设置 `PEERADDR` 环境变量。确保不包含开头的 `http://`。

3. 运行以下命令设置环境变量。

  ```
  export FABRIC_CFG_PATH=<full_path_to_config_folder>
  export CHANNEL=<CHANNEL_NAME>
  export CC_NAME=<CC_NAME>
  export CORE_PEER_ADDRESS=<PEERADDR>
  export ORDERER_1=<ORDERER_URL>
  export CORE_PEER_MSPCONFIGPATH=<PATH_TO_ADMIN_MSP>
  export CORE_PEER_TLS_ROOTCERT_FILE=<PATH_TO_PEER_TLS_CERT>
  export CORE_PEER_LOCALMSPID=<MSPID_OF_PEER_BEING_USED>
  ```
  {:codeblock}

  将相关字段替换为您自己的信息。
    - 将 `<full_path_to_config_folder>` 替换为[下载同级客户机](#peer-client)时创建的配置文件夹
    - 将 `<CHANNEL_NAME>` 替换为同级加入的通道的名称。
    - 将 `<CC_NAME>` 替换为要引用链代码的任何名称。
    - 将 `<PEERADDR>` 替换为上一步中当前正在使用的同级的同级端点。
    - 将 `<ORDERER_URL>` 替换为连接概要文件中排序节点的主机名和端口。
    - 将 `<PATH_TO_ADMIN_MSP>` 替换为同级管理员的 MSP 文件夹的路径。
    - 将 `<PATH_TO_PEER_TLS_CERT>` 替换为下载的同级 TLS 证书的路径。
    - 将 `<MSPID_OF_PEER_BEING_USED>` 替换为要用于访存起源区块、加入通道或安装链代码的同级的组织 MSPID。

  例如：

  ```
  export FABRIC_CFG_PATH=$HOME/config
  export CHANNEL=defaultchannel
  export CC_NAME=mycc
  export CORE_PEER_ADDRESS=9.30.94.174:30159
  export ORDERER_1=ash-zbc07b.4.secure.blockchain.ibm.com:21239
  export CORE_PEER_MSPCONFIGPATH=$HOME/fabric-ca-client/peer-admin/msp/
  export CORE_PEER_TLS_ROOTCERT_FILE=$HOME/fabric-ca-client/peer-tls/peertls.pem
  export CORE_PEER_LOCALMSPID=org1
  ```
  {:codeblock}

  您还希望启用 TLS，为此可以发出以下命令：

  ```
  export CORE_PEER_TLS_ENABLED=true
  ```
  {:codeblock}


### 使用 CLI 将同级加入通道
{: #icp-cli-join-peer-to-channel}

需要通过下列其中一个方法将组织添加到网络中的通道后，才能运行 CLI 命令将同级加入通道。

  - 可以为同级启动新的通道。作为通道启动者，您可以在[通道创建](/docs/services/blockchain/create_channel.html#creating-a-channel)期间自动包含您的组织。
  - 区块链网络的其他成员也可以使用[通道更新](/docs/services/blockchain/create_channel.html#updating-a-channel)将您的组织添加到现有通道。

    将组织添加到通道后，需要将同级的 signCert 添加到该通道，以便其他成员可以在交易期间验证您的数字签名。在安装期间，同级会上传其 signCert，这样您只需要将该证书同步到通道即可。在“网络监视器”的“通道”屏幕中，找到组织加入的通道，然后从**操作**标题下的下拉列表中选择**同步证书**。此操作会同步通道上所有同级的证书。

    **注：**仅当连接到入门套餐或企业套餐网络的 ICP 同级与使用“网络监视器”添加的其他同级属于同一组织时，才能在“网络监视器”的“通道”屏幕中查看添加到通道的新区块、已实例化的链代码以及其他通道更新。这是因为“网络监视器”是在“通道”屏幕上收集同级的信息，因此无法查看 {{site.data.keyword.cloud_notm}} 外部的同级。如果没有任何同级使用“专用数据”功能，那么“网络监视器”中针对组织中的不同同级的信息相同。

1. 确保已[在 CLI 中设置环境变量](#environment-variables)。

2. 访存通道的起源区块以在同级上构建通道分类帐。请注意，`$HOME/fabric-ca-client/tls-ibp/tls.pem` 表示入门套餐或企业套餐 TLS 证书的路径。如果路径不同，请确保设置正确的路径。

  ```
  peer channel fetch 0 -o ${ORDERER_1} -c ${CHANNEL} --cafile $HOME/fabric-ca-client/tls-ibp/tls.pem --tls
  ```
  {:codeblock}

  **注：**运行其中任一 CLI 命令时，可能会遇到以下警告，您可以安全地忽略此警告。

  ```
[msp] getPemMaterialFromDir -> WARN 001 Failed reading file
     /mnt/crypto/peer/peer/msp/intermediatecerts/<intermediate cert name>.pem: no pem content for file  /mnt/crypto/peer/peer/msp/intermediatecerts/<intermediate cert name>.pem
     ```

  运行以下命令来验证是否已成功将起源区块添加到同级容器：

  ```
  ls *.block
  ```
  {:codeblock}

  您看到类似于以下示例的内容时，即知道成功添加了起源区块：

  ```
  defaultchannel_0.block
  ```
  {:codeblock}

3. 访存通道的起源区块后，可以使用以下命令将同级加入通道：

  ```
  peer channel join -b ${CHANNEL}_0.block --tls
  ```
  {:codeblock}

  此命令成功执行后，应该会看到类似于以下示例的信息：

  ```
  2018-07-06 18:32:09.509 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
  2018-07-06 18:32:09.992 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
  2018-07-06 18:32:09.992 UTC [main] main -> INFO 003 Exiting.....
  ```

### 使用 CLI 在同级上安装链代码
{: #icp-toolcontainer-install-cc}

现在，我们已准备就绪，可以在同级上安装和实例化链代码。在以下指示信息中，我们将安装 `fabric-samples` 存储库中的 `fabcar` 链代码。请确保已事先[配置 GOPATH ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/dev-setup/devenv.html?highlight=gopath#set-your-gopath "设置 GOPATH")，然后使用以下命令从 GitHub 下载 `fabric-samples` 链代码：

```
cd $GOPATH/src
git clone https://github.com/hyperledger/fabric-samples
```
{:codeblock}

运行以下同级 CLI 命令将 `fabcar` 链代码安装到同级上：

```
peer chaincode install -n ${CC_NAME} -v v0 -p fabric-samples/chaincode/fabcar/go/
```
{:codeblock}

此命令成功完成后，应该会看到类似于下面的内容：

```
  2018-07-06 18:39:00.461 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
  2018-07-06 18:39:00.461 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
  2018-07-06 18:39:01.142 UTC [main] main -> INFO 003 Exiting.....
  ```

**注：**如果已在 IBP 入门套餐或企业套餐中运行的其他同级上安装并实例化了此链代码，那么在此同级上安装链代码之前，必须执行额外的步骤。请参阅此[故障诊断主题](#peer-ibp-troubleshooting)，以获取有关如何避免与此链代码安装方式关联的错误的更多指示信息。

### 使用 CLI 在通道上实例化链代码
{: #icp-toolcontainer-instantiate-cc}

因为只需要一个同级在通道上实例化链代码，所以不必使用您的同级来实例化链代码。在 {{site.data.keyword.blockchainfull_notm}} Platform 上托管的同级可以执行此操作。但是，如果您希望同级将链代码实例化到通道，那么可以通过运行以下命令来实现。确保已将 `CORE_PEER_TLS_ROOTCERT_FILE` 的值设置为入门套餐或企业套餐上 CA 的 TLS 证书的路径。

```
peer chaincode instantiate -o ${ORDERER_1} -C ${CHANNEL} -n ${CC_NAME} -v v0 -c '{"Args":[""]}' --tls --cafile $HOME/fabric-ca-client/tls-ibp/tls.pem -P ""
```
{:codeblock}

此命令成功完成后，应该会看到类似于下面的内容：

```
2018-07-06 18:43:15.066 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-07-06 18:43:15.066 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
```

实例化链代码后，可以使用链代码 query 和 invoke 命令在通道分类帐上读取和写入数据。有关更多信息，请参阅 Hyperledger Fabric 文档中的 [peer chaincode ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/latest/commands/peerchaincode.html) 命令。您需要使用代理 IP 和外部排序节点端口将排序节点端点传递到 invoke 命令。对于 query 命令，只需要传递同级端点。

## 更新链代码

一段时间后，您可能需要修改在同级上运行的链代码。由于链代码是安装在同级上并在通道上实例化的，因此需要更新通道上所有同级上的链代码。

完成以下步骤以更新链代码：

1. 要在每个同级上更新链代码，只需使用客户机应用程序或 CLI 命令重新运行用于在同级上安装链代码的进程即可。请确保指定的链代码名称与最初使用的链代码名称相同。但是，这次将链代码`版本`递增。所有同级都需要使用相同的链代码名称和版本。

2. 在通道上的所有同级上安装新的链代码后，使用“网络监视器”或[同级链代码升级 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/peerchaincode.html#peer-chaincode-upgrade) 命令来更新通道，以使用新的链代码。

有关使用“网络监视器”的“安装代码”面板更新通道上的链代码的更多信息，请参阅这些[指示信息](/docs/services/blockchain/install_instantiate_chaincode.html#updating-a-chaincode)中的步骤 2。

## 查看同级日志
{: #peer-ibp-view-logs}

在命令行中使用 [`kubectl CLI 命令`](#ca-kubectl-configure)或通过 ICP 集群中包含的 [Kibana ![外部链接图标](../images/external_link.svg "外部链接图")](https://www.elastic.co/products/kibana "您使用 Elasticsearch 的窗口")，可以查看组件日志。

- 使用 `kubectl logs` 命令来查看 pod 内的容器日志。如果不确定 pod 名称，请运行以下命令来查看 pod 的列表。

  ```
  kubectl get pods
  ```
  {:codeblock}

  然后，运行以下命令并将 `<pod_name>` 替换为上面命令输出中 pod 的名称，以检索位于 pod 内的同级容器的日志：

  ```
  kubectl logs <pod_name> -c peer
  ```
  {:codeblock}

  有关 `kubectl logs` 命令的更多信息，请参阅 [Kubernetes 文档 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs “Getting Started”)

- 或者，可以使用 [ICP 集群管理控制台](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/troubleshoot/events.html)来访问日志，这将在 Kibana 中打开日志。

   **注：**在 Kibana 中查看日志时，可能会收到响应 `No results found`。如果 ICP 将工作程序节点 IP 地址作为其主机名，那么会发生此情况。要解决此问题，请在面板顶部除去以 `node.hostname.keyword` 开头的过滤器，随后相关日志即可显示。

## 故障诊断
{: #peer-ibp-troubleshooting}

### **问题：**对同级运行 invoke 命令失败，错误为：`chaincode fingerprint mismatch`
{: #icp-cc-install-error}

对在 {{site.data.keyword.cloud_notm}} Private 中运行的同级运行 `peer chaincode invoke` 请求时，可能会收到 `chaincode fingerprint mismatch` 错误：

```
Error: Error endorsing invoke: rpc error: code = Unknown desc = error executing chaincode: could not get ChaincodeDeploymentSpec for marbles_rp:v0: get ChaincodeDeploymentSpec for marbles_rp/nancyremotepeer from LSCC error: chaincode fingerprint mismatch data mismatch -
```

如果运行链代码的同级在下列其中一个方面存在不一致，那么可能会导致此错误：

- 链代码名称
- 链代码版本
- 链代码路径
- 链代码二进制文件

如果使用“网络监视器 UI”在入门套餐或企业套餐中运行的同级上安装和实例化了链代码，然后在 {{site.data.keyword.cloud_notm}} Private 上运行的同级上安装该链代码，那么可能会发生此错误。`invoke` 请求会发生错误，因为同级上生成的链代码路径不同。

**解决方案：**如果要同时在 {{site.data.keyword.cloud_notm}}（例如，入门套餐或企业套餐）中和 {{site.data.keyword.cloud_notm}} Private 中的同级上运行链代码，请勿使用“网络监视器 UI”来安装链代码。请改为使用 [`peer chaincode package`![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/peerchaincode.html?highlight=peer%20chaincode%20package#peer-chaincode-package) 命令对链代码打包，然后通过运行 [`peer chaincode install`](#icp-toolcontainer-install-cc) 命令在所有同级上安装该包。

如果尝试在 {{site.data.keyword.cloud_notm}} Private 同级上安装链代码之前已在通道上安装并实例化了链代码，那么需要完成以下步骤以避免此问题：

1. 使用 [`peer chaincode package` ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/peerchaincode.html?highlight=peer%20chaincode%20package#peer-chaincode-package) 命令对链代码打包。
2. 通过运行 `peer chaincode install` 命令，在 {{site.data.keyword.cloud_notm}} Private 上运行的同级上安装该链代码包。
3. 如果您具有特定于平台的二进制文件，那么可以运行 [`peer chaincode upgrade` ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/peerchaincode.html?highlight=peer%20chaincode%20package#peer-chaincode-upgrade) 命令来升级在入门套餐或企业套餐同级上运行的链代码，该同级使用该链代码包。
4. 使用“网络监视器 UI”或 CLI 在通道上实例化新安装的链代码。

升级链代码的过程还可以在 Hyperledger Fabric 文档的 [`Chaincode for Operators` ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/chaincode4noah.html) 中找到。
