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

# 在使用多云网络的 {{site.data.keyword.cloud_notm}} Private 上操作同级
{: #peer-operate_icp}

***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

在 {{site.data.keyword.cloud_notm}} Private (ICP) 上设置 {{site.data.keyword.blockchainfull}} Platform 同级后，您需要完成若干操作步骤，同级才能发出事务来查询和调用区块链网络的分类帐。这些步骤包括将组织添加到通道，将同级连接到通道，在同级上安装链代码，在通道上实例化链代码，以及将应用程序连接到同级。如果要将同级连接到入门套餐或企业套餐网络，请参阅[在使用入门套餐或企业套餐的 {{site.data.keyword.cloud_notm}} Private 上操作同级](/docs/services/blockchain/peer_operate_ibp.html#peer-operate_icp)。
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

Fabric SDK 是建议的路径，不过指示信息假定您熟悉 SDK 的操作。如果首选使用命令行，那么可以使用 Fabric peer 二进制文件。

如果组织还不是联盟或通道的成员，那么可以使用这些步骤来[创建通道](#create-channel)。这些指示信息将指导您如何[准备组织定义](#organization-definition)。此定义将用于通过将您添加到排序节点系统通道，使您成为联盟的成员。在此之后，您将能够通过[创建通道事务](#peer-icp-channeltx)来构成新的通道。
<!--
It is recommended that you deploy at least two instances of the peer Helm chart for [high availability](/docs/services/blockchain/peer_icp.html#high-availability). Therefore, you need to follow these operations steps once for each peer. When you are ready to invoke and query chaincode from your application, connect to both peers to ensure that your [applications are highly available](/docs/services/blockchain/v10_application.html#ha-app).
-->

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

**注：**如果将同级部署在防火墙后，那么需要打开传递，以支持平台上的网络与同级完成 TlS 握手。您只需要对绑定到同级端口 7051 的 Node 端口启用传递。有关更多信息，请参阅[查找同级端点信息](/docs/services/blockchain/peer_operate_ibp.html#peer-endpoint)。

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
{: #peer-node-sdk}

同级已使用内部同级管理员的 signCert 进行部署。这将允许您使用同级管理员的证书和 MSP 文件夹来操作同级。

找到在[注册同级管理员](/docs/services/blockchain/CA_operate.html#enroll-admin)时创建的证书。如果使用的是示例命令，那么可以在 `$HOME/fabric-ca-client/peer-admin` 中找到同级管理员的 MSP 文件夹。
  - 可以使用 MSP 文件夹中的 signCert（公用密钥）和专用密钥通过 SDK 来构建同级管理员用户上下文。您可以在以下位置找到这些键：
    - signCert 可以在 **signcerts** 文件夹 `$HOME/fabric-ca-client/peer-admin/msp/signcerts` 中找到
    - 专用密钥可以在 **keystore:** 文件夹 `$HOME/fabric-ca-client/peer-admin/msp/keystore` 中找到
    有关如何仅使用公用和专用密钥来构造用户上下文并操作 SDK 的示例，请参阅[使用网络监视器生成证书](/docs/services/blockchain/v10_application.html#enroll-panel)。

<!-- You can also use the SDK to generate the peer admin signCert and private key using the endpoint information of CA on Starter Plan or Enterprise Plan and your [peer admin username and password](/docs/services/blockchain/CA_operate.html#register-admin). -->

### 将同级的 TLS 证书传递到 SDK
{: #icp-peer-download-tlscert}

您需要引用同级的 TLS 证书对来自 SDK 的通信进行认证。找到[从同级容器下载](#peer-tls)的 TLS 证书，并将其保存在应用程序可以对其进行引用的位置。如果使用的是示例命令，那么可以在 `$HOME/fabric-ca-client/peer-tls/peertls.pem` 中找到同级 TLS 证书。然后，可以使用简单的读取文件命令将该 TLS 证书导入应用程序。

```
var peerTLSCert = fs.readFileSync(path.join(__dirname, './peertls.pem'));
```
{:codeblock}

### 向 SDK 提供同级端点信息
{: #peer-SDK-endpoints}

找到[同级的端点信息](#peer-endpoint)，然后通过声明新的同级变量，向 SDK 提供这些信息。以下示例将同级定义为 Fabric 网络上的端点，并向其传递导入的 TLS 证书。

```
var peer = fabric_client.newPeer('grpcs://9.30.94.174:30159', { pem:  Buffer.from(peerTLSCert).toString(), 'ssl-target-name-override': null});
```
{:codeblock}

<!--
You need to specify a `ssl-target-name-override` of `<something>.blockchain.com` in order for the peer to resolve the DNS request.
-->

如果您有一个大型网络，其中有多个属于不同组织的同级需要对事务背书，那么您可能希望与联盟合作，以[构建公共连接概要文件 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/tutorial-network-config.html "common connection profile")

### 将排序节点的 TLS 证书传递到 SDK
{: #icp-orderer-download-tlscert}

您还需要联盟的排序节点的 TLS 证书，才能加入通道并提交事务。如果您是排序节点的管理员，请遵循相关指示信息[下载排序节点 TLS 证书](/docs/services/blockchain/orderer_operate.html#orderer-tls)。如果使用的是示例命令，那么可以在 `$HOME/fabric-ca-client/orderer-tls/orderertls.pem` 中找到同级 TLS 证书。如果排序节点由其他组织进行管理，那么他们需要在频带外操作中向您提供 TLS 证书。然后，您可以将 TLS 证书导入应用程序。

```
var ordererTLSCert = fs.readFileSync(path.join(__dirname, './orderertls.pem'));
```
{:codeblock}

### 向 SDK 提供排序节点信息
{: #orderer-SDK-endpoints}

要使用 SDK，您还需要联盟中排序节点的端点信息。如果您是排序节点的管理员，那么可以使用这些指示信息来[检索排序节点端点信息](/docs/services/blockchain/orderer_operate.html#orderer-endpoint)。如果排序节点由其他组织进行管理，那么他们需要在频带外操作中向您提供排序节点 URL。以下示例将排序节点定义为端点，并向其传递排序节点 TLS 证书。

```
var orderer = fabric_client.newOrderer('grpcs://9.30.94.174:30167', { pem:  Buffer.from(ordererTLSCert).toString(), 'ssl-target-name-override': null});
```
{:codeblock}
<!--
You need to specify a `ssl-target-name-override` of `<something>.blockchain.com` in order for the peer to resolve the DNS request.
-->

### 使用 SDK 加入通道
{: #peer-join-channel-sdk}

您的组织需要成为通道的成员，然后才能将同级加入通道。如果您不是通道的成员，那么可以遵循相关指示信息[创建新的通道](#create-channel)。

在组织成为通道的成员后，请遵循相关指示信息使用 SDK [将同级加入通道](/docs/services/blockchain/v10_application.html#join-channel-sdk)。

### 使用 SDK 在同级上安装链代码
{: #peer-install-cc-sdk}

使用以下指示信息通过 SDK 在同级上[安装链代码](/docs/services/blockchain/v10_application.html#install-cc-sdk)。

### 使用 SDK 在通道上实例化链代码
{: #peer-instantiate-cc-sdk}

只需要通道中的一个成员对链代码进行实例化或更新。因此，在其同级上安装了链代码的任何通道成员都可以实例化链代码并指定背书策略。但是，如果要使用同级在通道上实例化链代码，那么可以使用 SDK 并遵循指示信息来[实例化链代码](/docs/services/blockchain/v10_application.html#instantiate-cc-sdk)。

## 使用 CLI 来操作同级
{: #peer-cli-operate}

您还可以通过命令行使用 Fabric Peer 二进制文件来操作同级。

同级已使用内部同级管理员的 signCert 进行部署，允许该身份操作同级。以下指示信息将使用在[部署同级](/docs/services/blockchain/CA_operate.html#register-admin)时生成的同级管理员的 MSP 文件夹来将同级加入通道，在同级上安装链代码，然后在通道上实例化链代码。

### 下载 Fabric 同级客户机
{: #peer-client}

要通过远程客户机与同级进行交互，您需要下载 [Fabric 同级客户机 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/peercommand.html "Fabric peer 命令")。

获取同级客户机的最简单方法是从 Hyperledger 项目下载 Fabric 工具二进制文件。使用命令行创建要将二进制文件下载到的目录，然后通过发出以下命令来访存这些二进制文件。您需要首先安装 [Curl ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/prereqs.html#install-curl "Curl")。

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

下载二进制文件将创建包含 core.yaml、orderer.yaml 和 configtx.yaml 文件的 config 文件夹。将 Fabric 配置路径设置为此 config 目录：

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

需要在本地计算机上对证书执行一些管理工作，才能操作同级。您还需要确保可以从同级访问 TLS 证书。有关要使用的证书的更多信息，请参阅[在 {{site.data.keyword.cloud_notm}} Private 上操作认证中心](/docs/services/blockchain/CA_operate.html)中的[成员资格服务提供者](/docs/services/blockchain/CA_operate.html#msp)。

1. 将同级管理员的 signCert 移至名为 `admincerts` 的新文件夹：

    ```
    cd $HOME/fabric-ca-client/peer-admin/msp
    mkdir admincerts
    cp signcerts/cert.pem admincerts/cert.pem
    ```
    {:codeblock}

2. 确保您已[下载同级 TLS 证书](#peer-tls)，并且可以通过命令行引用该证书。如果遵循的是本文档中的示例命令，那么可以在 `$HOME/fabric-ca-client/peer-tls/peertls.pem` 文件中找到此 TLS 证书。

3. 您还需要引用排序节点的 TLS 证书。如果您是排序节点的管理员，请遵循相关指示信息[下载排序节点 TLS 证书](/docs/services/blockchain/orderer_operate.html#orderer-tls)。如果排序节点由其他组织进行管理，那么他们需要在频带外操作中向您提供 TLS 证书。将此证书保存在可在未来命令中引用此证书的位置。

您可以运行 tree 命令来验证是否已完成这些步骤。浏览至存储证书的目录。tree 命令应该生成类似于以下结构的结果：
```
cd $HOME/fabric-ca-client
tree
.
├── ca-admin
│   ├── fabric-ca-client-config.yaml
│   └── msp
│       ├── cacerts
│       │   └── 9-30-250-70-30395-SampleOrgCA.pem
│       ├── keystore
│       │   └── 2a97952445b38a6e0a14db134645981b74a3f93992d9ddac54cb4b4e19cdf525_sk
│       ├── signcerts
│       │   └── cert.pem
│       └── user
├── catls
│   └── tls.pem
├── orderer-tls
│   └── orderertls.pem
├── peer-admin
│   └── msp
│       ├── admincerts
│       │   └── cert.pem
│       ├── cacerts
│       │   └── 9-30-250-70-30395-SampleOrgCA.pem
│       ├── keystore
│       │   └── 24f76217c5c7e641ee29b068712181294e427cbee4dbfce230a1a98b53489cbe_sk
│       ├── signcerts
│       │   └── cert.pem
│       └── user
├── peer-tls
│   └── peertls.pem
└── tlsca-admin
    ├── fabric-ca-client-config.yaml
    └── msp
        ├── cacerts
        │   └── 9-30-250-70-30395-tlsca.pem
        ├── keystore
        │   └── 45a7838b1a91ddfe3d4d22a5a7f2639b868493bcce594af3e3ceb9c07899d117_sk
        ├── signcerts
        │   └── cert.pem
        └── user
```

### 设置 CLI 环境变量
{: #environment-variables}

将所有证书都移至必需的位置后，需要设置一些环境变量供同级 CLI 命令使用。然后，即准备就绪可以使用 Fabric 同级客户机来操作同级。

1. 确保已[在 CLI 中设置环境变量](#environment-variables)。

2. 将 Fabric 配置路径设置为[下载 Fabric 工具二进制文件](#peer-client)时创建的此 config 目录：

    ```
    export FABRIC_CFG_PATH=<full_path_to_config_folder>
    ```
    {:codeblock}

    通过设置此变量，可以在任何目录中使用同级客户机来运行命令。

3. 您需要排序节点的端点信息。如果您是排序节点的管理员，那么可以使用这些指示信息来[检索排序节点端点信息](/docs/services/blockchain/orderer_operate.html#orderer-endpoint)。如果排序节点由其他组织进行管理，那么他们需要在频带外操作中向您提供排序节点 URL。

4. [检索同级端点信息](#peer-endpoint)。我们将使用此 URL 来设置 `PEERADDR` 环境变量。您需要去除开头的 `http://`。

5. 运行以下命令设置环境变量：

  ```
  export CHANNEL=<CHANNEL_NAME>
  export CC_NAME=<CC_NAME>
  export CORE_PEER_ADDRESS=<PEERADDR>
  export ORDERER_1=<ORDERER_URL>
  export CORE_PEER_MSPCONFIGPATH=<PATH_TO_ADMIN_MSP>
  export CORE_PEER_LOCALMSPID=<MSPID_OF_PEER_BEING_USED>
  ```
  {:codeblock}

  将相关字段替换为您自己的信息。
    - 将 `<CHANNEL_NAME>` 替换为同级加入的通道的名称。
    - 将 `<CC_NAME>` 替换为要引用链代码的任何名称。
    - 将 `<PEERADDR>` 替换为上一步中同级端点的主机名和端口。
    - 将 `<ORDERER_URL>` 替换为联盟排序节点的主机名和端口。
    - 将 `<PATH_TO_ADMIN_MSP>` 替换为同级管理员的 MSP 文件夹的路径。
    - 将 `<MSPID_OF_PEER_BEING_USED>` 替换为要用于访存起源区块、加入通道或安装链代码的同级的组织 MSPID。此值在 Helm 图表部署期间提供。

  例如：

  ```
  export CHANNEL=mychannel
  export CC_NAME=mycc
  export CORE_PEER_ADDRESS=9.30.250.70:32110
  export ORDERER_1=9.30.250.70:31507
  export CORE_PEER_MSPCONFIGPATH=$HOME/fabric-ca-client/peer-admin/msp/
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

您的组织需要成为通道的成员，然后才能将同级加入通道。如果您不是通道的成员，那么可以遵循相关指示信息[创建新的通道](#create-channel)。

1. 确保已[在 CLI 中设置环境变量](#environment-variables)。

2. 访存通道的起源区块以在同级上构建通道分类帐。将 `CORE_PEER_TLS_ROOTCERT_FILE` 设置为排序节点 TLS 证书的路径，然后运行以下命令：

  ```
  peer channel fetch 0 -o ${ORDERER_1} -c ${CHANNEL} --tls --cafile $HOME/fabric-ca-client/orderer-tls/orderertls.pem
  ```
  {:codeblock}

  如果此命令成功，您将看到类似于以下内容的结果：
  ```
  2018-11-27 17:00:49.387 EST [cli/common] readBlock -> INFO 041 Received block: 0
  ```

  **注：**运行其中任一 CLI 命令时，可能会遇到以下警告，您可以安全地忽略此警告。

  ```
[msp] getPemMaterialFromDir -> WARN 001 Failed reading file
     /mnt/crypto/peer/peer/msp/intermediatecerts/<intermediate cert name>.pem: no pem content for file  /mnt/crypto/peer/peer/msp/intermediatecerts/<intermediate cert name>.pem
     ```

{:codeblock}

  运行以下命令来验证是否已成功将起源区块添加到同级容器：

  ```
  ls *.block
  ```
  {:codeblock}

  您看到类似于以下示例的内容时，即知道成功添加了起源区块：

  ```
  defaultchannel_0.block
  ```

3. 访存通道的起源区块后，可以将同级加入通道。使用以下命令将 `CORE_PEER_TLS_ROOTCERT_FILE` 设置为同级 TLS 证书的路径，并加入通道：

  ```
  export CORE_PEER_TLS_ROOTCERT_FILE=$HOME/fabric-ca-client/peer-tls/peertls.pem
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

运行以下同级 CLI 命令将 `fabcar` 链代码安装到同级上。将 `CORE_PEER_TLS_ROOTCERT_FILE` 设置为同级 TLS 证书的路径。

  ```
  export CORE_PEER_TLS_ROOTCERT_FILE=$HOME/fabric-ca-client/peer-tls/peertls.pem
  peer chaincode install -n ${CC_NAME} -v v0 -p fabric-samples/chaincode/fabcar/go/
  ```
  {:codeblock}

  此命令成功完成后，应该会看到类似于下面的内容：

  ```
  2018-11-27 17:05:54.062 EST [chaincodeCmd] install -> INFO 050 Installed remotely response:<status:200 payload:"OK"
  ```

<!--
**Note:** If this chaincode has already been installed and instantiated on another peer running in IBP Starter or Enterprise Plan, there are additional steps that must be performed before installing the chaincode on the peer. For more instructions about how to avoid errors that are associated with how this chaincode is installed, see the [troubleshooting topic](#icp-cc-install-error).
-->

### 使用 CLI 在通道上实例化链代码
{: #icp-toolcontainer-instantiate-cc}

因为只需要一个同级在通道上实例化链代码，所以不必使用您的同级来实例化链代码。在 {{site.data.keyword.blockchainfull_notm}} Platform 上托管的同级可以执行此操作。但是，如果您希望同级将链代码实例化到通道，那么可以通过运行以下命令来实现。将 `CORE_PEER_TLS_ROOTCERT_FILE` 的值设置为同级 TLS 证书的路径。将 `--cafile` 的值设置为排序节点 TLS 证书。

```
cd $HOME/fabric-ca-client
export CORE_PEER_TLS_ROOTCERT_FILE=$HOME/fabric-ca-client/peer-tls/peertls.pem
peer chaincode instantiate -o ${ORDERER_1} -C ${CHANNEL} -n ${CC_NAME} -v v0 -c '{"Args":[""]}' --tls --cafile $PWD/orderer-tls/orderertls.pem -P ""
```
{:codeblock}

此命令成功完成后，应该会看到类似于下面的内容：

```
2018-11-27 17:09:28.013 EST [chaincodeCmd] checkChaincodeCmdParams -> INFO 048 Using default escc
2018-11-27 17:09:28.013 EST [chaincodeCmd] checkChaincodeCmdParams -> INFO 049 Using default vscc
```

实例化链代码后，可以使用链代码 query 和 invoke 命令在通道分类帐上读取和写入数据。有关更多信息，请参阅 Hyperledger Fabric 文档中的 [peer chaincode ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/latest/commands/peerchaincode.html) 命令。您需要使用代理 IP 和外部排序节点端口将排序节点端点传递到 invoke 命令。对于 query 命令，只需要传递同级端点。

## 在 {{site.data.keyword.cloud_notm}} Private 中查看同级日志
{: #peer-log-icp}

您可以通过 ICP 控制台访问同级日志，并在 Kibana 界面中检查日志。

1. 在 ICP 控制台中，单击左上角的**菜单**图标。
2. 从菜单列表中，单击**工作负载** > **Helm 发布**。
3. 在“Helm 发布”表中，单击您的 Helm 发布的名称。
4. 向下滚动到 **Pod** 部分，然后单击表行末尾的**查看日志**链接。这将在 Kibana 界面中打开同级日志。

## 更新链代码

一段时间后，您可能需要修改在同级上运行的链代码。由于链代码是安装在同级上并在通道上实例化的，因此需要更新通道上所有同级上的链代码。

完成以下步骤以更新链代码：

1. 要在每个同级上更新链代码，只需使用客户机应用程序或 CLI 命令重新运行用于在同级上安装链代码的进程即可。请确保指定的链代码名称与最初使用的链代码名称相同。但是，这次将链代码`版本`递增。所有同级都需要使用相同的链代码名称和版本。

2. 在通道中的所有同级上安装新的链代码后，使用 [peer chaincode upgrade ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/peerchaincode.html#peer-chaincode-upgrade) 命令来更新通道，以使用新的链代码。

## 重新启动在 ICP 中运行的同级
{: #peer-restart}

每当向同级添加证书后，都需要重新启动同级节点，以便该节点识别到新证书。

可以使用 **kubectl** 命令来重新启动在 ICP 中运行的同级。该命令需要同级 **pod** 及其**容器**的名称作为命令参数。有关使用 kubectl 命令的更多信息，请参阅[配置 kubectl CLI](#peer-kubectl-configure)。

1. 在 ICP 控制台中，找到用于同级部署的 **pod** 名称和**容器**名称。

  1. 单击控制台左上角的菜单图标，然后单击**工作负载 > 部署**。
  2. 在表中找到并单击同级发布以将其打开。
  3. 向下滚动以查看关联的 **pod** 名称。记下 **pod** 名称。
  4. 单击该 pod 以将其打开。
  5. 单击**容器**选项卡。记下同级**容器**名称。

2. 在命令行中，运行以下命令来重新启动同级，并将 `<PEER_POD_NAME>` 和 `<PEER_CONTAINER_NAME>` 替换为上面的值。

  ```
  kubectl exec <PEER_POD_NAME> -c <PEER_CONTAINER_NAME> /usr/sbin/killall5
  ```
  {:codeblock}

3. 可以通过运行 `kubectl get pods` 命令并检查输出中 pod 的 **RESTART** 计数，验证同级是否已重新启动。

## 创建通道
{: #create-channel}

如果组织还不是联盟或通道的成员，那么可以使用以下步骤来创建通道。您还可以使用这些步骤来更新现有通道。这些指示信息将指导您如何[准备组织定义](#organization-definition)。此定义将用于通过将您添加到排序节点系统通道，使您成为[联盟的成员](orderer_operate.html#add-organizations-to-consortium)。在此之后，您将能够通过[创建通道事务](#peer-icp-channeltx)来构成新的通道。

如果您已加入联盟，那么只需完成准备通道事务的步骤。联盟的其他成员也可以将组织作为成员来构成新的通道。您可以在需要向联盟发送更新（例如定义新的锚点同级，添加新的管理员证书或更新加密资料）时更新组织定义。

### 下载 configtxgen 工具
{: #peer-icp-configtxgen}

如果组织需要加入联盟或通道，那么您需要下载 [configtxgen ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/configtxgen.html "configtxgen") 工具。

获取 configtxgen 的最简单方法是从 Hyperledger 项目下载所有 Fabric 工具二进制文件。使用命令行浏览至要将二进制文件下载到的目录，然后通过发出以下命令来访存这些二进制文件。您需要首先安装 [Curl ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/prereqs.html#install-curl "Curl")。

```
curl -sSL http://bit.ly/2ysbOFE | bash -s 1.2.1 1.2.1 -d -s
```
{:codeblock}

此命令将在 `bin/` 目录中安装 configtxgen。将 `PATH` 路径设置为 Fabric 工具二进制文件的下载目录：

```
export PATH=$PATH:<full_path_to_configtxgen>/bin
```
{:codeblock}

例如，如果已将二进制文件安装在主目录中，那么应将 `PATH` 设置为：

```
export PATH=$PATH:$HOME/bin
```

## 准备组织定义
{: #organization-definition}

部署了组件后，组织可以通过准备组织定义文件来加入联盟。定义包含参与联盟管理所需的所有信息，例如创建或加入新的通道，向通道添加同级或者实例化链代码。定义还包含组织名称、MSPID 和 MSP 证书。所有组织都需要完成此步骤。

### 准备加密资料

准备组织定义之前，需要注册和登记[同级的管理员](/docs/services/blockchain/CA_operate.html#register-admin)。浏览至在其中创建同级管理员的 MSP 文件夹的目录。示例步骤在 `$HOME/fabric-ca-client/peer-admin/msp` 中创建了此文件夹。您需要在此文件夹中执行其他一些步骤，然后 `configtxgen` 工具才能使用 MSP。

1. 从 CA 复制 TLS 证书，并将其放入名为 `tlscacerts` 的新文件夹中。

  ```
  cd $HOME/fabric-ca-client/peer-admin/msp
  mkdir tlscacerts
  ```

  然后，需要将证书复制到所创建的 `tlscacerts` 目录：

  ```
  cp $HOME/fabric-ca-client/tlsca-admin/cacerts/<xxxx>tlsca.pem tlscacerts/
  ```

  命令可能类似于以下内容：

  ```
  cp fabric-ca-client/tlsca-admin/cacerts/9-30-250-70-32129-tlsca.pem /tlscacerts/
  ```
  {:codeblock}

2. 将 signCert 从 `signcerts` 文件夹移至名为 `admincerts` 的新文件夹。

  ```
  cd $HOME/fabric-ca-client/peer-admin/msp
  mkdir admincerts
  cp signcerts/cert.pem admincerts/cert.pem
  ```
  {:codeblock}

`configtxgen` 工具通过使用 configtx.yaml 文件来创建定义。您可以在下面找到此文件的样本版本：

```
---
################################################################################
#
#   Section: Organizations
#
#   - This section defines the different organizational identities which will
#   be referenced later in the configuration.
#
################################################################################
Organizations:
    - &org1
        # DefaultOrg defines the organization which is used in the sampleconfig
        # of the fabric.git development environment
        Name: org1

        # ID to load the MSP definition as
        ID: org1

        MSPDir: <path_to_peer-admin>/msp

        AnchorPeers:
            # AnchorPeers defines the location of peers which can be used
            # for cross org gossip communication.  Note, this value is only
            # encoded in the genesis block in the Application section context
            - Host: 9.30.250.70
              Port: 30481
```
{:codeblock}

此文件包含用于在联盟内定义组织的信息。在[下载的 Fabric 同级客户机](/docs/services/blockchain/peer_operate_icp.html#peer-client)的 `/config` 文件夹中还提供了此文件的更复杂版本。您可以选择编辑该文件，也可以将其替换为以上样本。请记下此 `/config` 文件夹的位置，以设置下面的 `FABRIC_CFG_PATH` 的值。编辑此文件的 `Organizations` 部分，并设置以下值：

- `Name:` 可以是您希望用于组织的任何名称。

- `ID:` 是 MSPID 的值，这会成为组织的唯一标识。此组织 MSP 值是在部署同级 Helm 图表时指定的。还可以通过运行以下命令，并将 `<peer pod name>` 替换为同级的 pod 的值，在同级容器中显示此值：

  ```
  kubectl exec -it <peer pod name> -c peer sh
  $ env | grep CORE_PEER_LOCALMSPID
  ```
  {:codeblock}

  输出可能类似于以下内容：
  ```
  CORE_PEER_LOCALMSPID=org1
  ```

  因此，MSP `ID` 是 org1。

- `MSPDir:` 是管理员 MSP 的标准路径，例如：`/Users/chandra/fabric-ca-client/peer-admin/msp`

- `AnchorPeers:` 字段是组织选择作为同级负责人，负责通过 Gossip 进行组织间通信的同级。这用于专用数据或服务发现等功能；但是，同级 Helm 图表当前不支持专用数据和服务发现，尽管仍然需要该字段。`Host` 和 `Port` 值可在 ICP 同级 Helm 发布页面中的“注释”部分 `#1. 通过运行以下命令来获取应用程序 URL` 下获取。输出将类似于以下内容：

```
http://9.30.250.70:30481
```

其中，`Host:` 为 `9.30.250.70`，`Port:` 为 `30481`

然后使用以下命令显示组织定义：

```
export FABRIC_CFG_PATH=<path_to_config_folder>
configtxgen -printOrg <org_name> > <path_to_org_definition>orgdefinition.json
```
{:codeblock}

您的调用可能类似于以下示例命令：

```
export FABRIC_CFG_PATH==<full_path_to_config_folder>
mkdir -p $HOME/fabric-ca-client/org-definitions
configtxgen -printOrg org1 > $HOME/fabric-ca-client/org-definitions/org1definition.json
```
{:codeblock}

如果此命令成功，那么 `configtxgen` 将显示 JSON 格式的组织定义。您需要在频带外操作中将此文件发送到排序节点组织以加入联盟。然后，排序节点组织可以通过将定义添加到系统通道来[构成联盟或添加到现有联盟](/docs/services/blockchain/orderer_operate.html#consortium)，从而允许您创建新的通道以及由其他联盟成员将您添加到通道。

## 创建通道事务
{: #peer-icp-channeltx}

您的组织应该已准备好[组织定义](#organization-definition)并成为联盟的成员，然后您才能创建新的渠道。如果需要[构成联盟或添加到联盟](/docs/services/blockchain/orderer_operate.html#consortium)，请遵循以下指示信息进行操作。如果联盟成员的组织已经添加到系统通道，那么还可以轻松地将这些成员添加到新的通道。对于不属于系统通道成员的组织，只能通过使用[通道更新请求 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/channel_update_tutorial.html) 将其组织定义添加到通道，从而手动将这些组织加入通道。您还可以使用这些步骤来更新现有通道。

### 构成新通道
{: #peer-icp-create-channel}

要构成新的通道，组织需要使用 [configtxgen 工具 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/configtxgen.html "configtxgen") 来创建通道创建事务建议。此工具使用 `configtx.yaml` 文件来定义新通道成员。下面提供了样本 `configtx.yaml` 文件。在[下载的 Fabric 同级客户机](peer_operate_icp.html#peer-client)的 `/config` 文件夹中还提供了此文件的更复杂版本。您可以选择编辑该文件，也可以将其替换为样本。请记下此 `/config` 文件夹的位置，以设置下面的 `FABRIC_CFG_PATH` 的值。
```
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#
---
################################################################################
#
#   Section: Organizations
#
#   - This section defines the different organizational identities which will
#   be referenced later in the configuration.
#
################################################################################
Organizations:
    - &org1

        Name: org1

        # ID to load the MSP definition as
        ID: org1

        MSPDir: /Users/chandra/fabric-ca-client/peer-admin/msp

        AnchorPeers:

            - Host: 9.30.250.70
              Port: 32110


################################################################################
#
#   SECTION: Capabilities
#
################################################################################
Capabilities:
    # Channel capabilities apply to both the orderers and the peers and must be
    # supported by both.  Set the value of the capability to true to require it.
    Global: &ChannelCapabilities

        V1_1: true

    # Orderer capabilities apply only to the orderers, and may be safely
    # manipulated without concern for upgrading peers.  Set the value of the
    # capability to true to require it.
    Orderer: &OrdererCapabilities

        V1_1: true

    # Application capabilities apply only to the peer network, and may be safely
    # manipulated without concern for upgrading orderers.  Set the value of the
    # capability to true to require it.
    Application: &ApplicationCapabilities

        V1_2: true

################################################################################
#
#   SECTION: Application
#
#   - This section defines the values to encode into a config transaction or
#   genesis block for application related parameters
#
################################################################################

Application: &ApplicationDefaults

    # Organizations is the list of orgs which are defined as participants on
    # the application side of the network
    Organizations:

    Capabilities:
        <<: *ApplicationCapabilities


################################################################################
#
#   Profile
#
#   - Different configuration profiles may be encoded here to be specified
#   as parameters to the configtxgen tool
#
################################################################################

Profiles:

    mychannel:
        Consortium: SampleConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *org1
            Capabilities:
                <<: *ApplicationCapabilities

```
{:codeblock}

与创建新通道相关的三个部分是 **Organizations**、**Capabilities** 和 **Profiles**：

- **Organizations:** 此部分定义了联盟的所有成员。每个组织都有一个锚点，例如 `&Org1`。在此锚点下，可以找到组织的名称、MSPID、其 MSP 文件夹的目录以及其组织的锚点同级，以用于跨同级 Gossip 通信。您可以通过以下步骤填写每个联盟成员的组织概要文件：
  1. 使用组织的 MSPID 指定组织的`名称`和`标识`。创建通道的组织需要知道部署同级 Helm 图表时指定的 MSPID。
  2. 在 `MSPDir` 中，指定用于创建[组织定义](#organization-definition)的 MSP 文件夹的标准路径。请注意，通道创建事务中不会使用任何加密资料。*configtxgen* 工具将忽略 MSP 的实际内容。但是，它需要在该位置有一个 MSP 文件夹，并且具有正确的文件夹子结构。
  3. `AnchorPeers` 参数用于标识每个组织用于通过 Gossip 进行组织间通信的同级负责人的身份。指定将充当此组织的[锚点同级 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/glossary.html) 的同级的主机名和端口。此值对于使用专用数据或服务发现等功能非常重要；但是，同级 Helm 图表当前不支持专用数据和服务发现。

- **Capabilities:** 此部分需要位于 `configx.yaml` 中，但并不需要对其进行任何更改。

- **Profiles:** 部分。此部分包含创建新通道所需的信息。profile 包含以下信息：
  1. 新通道的名称。
  2. 在系统通道内定义的联盟的名称，此名称将用于创建通道。
  3. 将添加到通道的组织的列表。列出的组织名称将使用 `Organizations` 部分中列出的锚点同级来查找列出的组织 MSPID 和关联的锚点同级。

  *configtxgen* 工具使用此 `Profiles` 部分来创建通道创建建议。

更新了配置文件之后，请使用以下命令生成通道更新事务建议：

``` 
export FABRIC_CFG_PATH=<path_to_config_folder>
configtxgen -profile <channel_profile_name> -outputCreateChannelTx ./<channel_profile_name>.tx -channelID <channel_profile_name>
```
{:codeblock}

实际命令将类似于以下示例：
``` 
export FABRIC_CFG_PATH=$HOME/config
configtxgen -profile mychannel -outputCreateChannelTx ./mychannel.tx -channelID mychannel
```
{:codeblock}


如果此命令成功，您将看到类似于以下输出的结果。此命令将在 `FABRIC_CFG_PATH` 目录中写入通道更新事务建议。

```
2018-11-27 16:39:36.130 EST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-11-27 16:39:36.134 EST [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
2018-11-27 16:39:36.134 EST [common/tools/configtxgen/encoder] NewApplicationGroup -> WARN 003 Default policy emission is deprecated, please include policy specificiations for the application group in configtx.yaml
2018-11-27 16:39:36.135 EST [common/tools/configtxgen/encoder] NewApplicationOrgGroup -> WARN 004 Default policy emission is deprecated, please include policy specificiations for the application org group org1 in configtx.yaml
2018-11-27 16:39:36.136 EST [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 005 Writing new channel tx
```

如果背书策略不需要联盟的其他成员的额外签名，可以将建议直接提交给排序节点以构成新的通道。在生成通道更新事务以创建通道的位置运行以下命令。

```
export CORE_PEER_LOCALMSPID=<ORG_MSPID>
export CORE_PEER_MSPCONFIGPATH=<PATH_TO_ADMIN_MSP>
peer channel create -o <orderer_url> --tls --cafile <orderer_tls_cert> -c <channel_profile_name> -f ./<channel_profile_name>.tx
```
{:codeblock}

其中，`<ORG_MSPID>` 是组织的 MSPID，`<PATH_TO_ADMIN_MSP>` 是同级管理员的 MSP 文件夹的路径。在命令中，`<orderer_url>` 是将创建通道的排序节点的 URL，`<orderer_tls_cert>` 是排序节点 TLS 证书的路径。实际命令可能类似于以下示例：
```
export CORE_PEER_LOCALMSPID=org1
export CORE_PEER_MSPCONFIGPATH=$HOME/fabric-ca-client/peer-admin/msp/
peer channel create -o 9.30.250.70:31507 --tls --cafile $PWD/orderer-tls/orderertls.pem -c mychannel -f ./mychannel.tx
```
{:codeblock}

```
2018-11-27 16:59:30.610 EST [cli/common] readBlock -> INFO 04f Received block: 0
```


## 对建议背书并提交请求

数量达到系统通道背书策略要求的组织对新的通道创建建议签名后，可以将通道更新事务提交给排序节点以构成新的通道。

可以使用 Node Fabric SDK API 在一个事务中完成签名和提交。有关更多信息，请访问 Node SDK 文档中的 [How to create a Hyperledger Fabric Channel ![外部链接图标](../images/external_link.svg "外部链接图标")]](https://fabric-sdk-node.github.io/release-1.3/tutorial-channel-create.html) 教程。

## 查看同级日志
{: #peer-icp-view-logs}

在命令行中使用 [`kubectl CLI 命令`](#peer-kubectl-configure)或通过 ICP 集群中包含的 [Kibana ![外部链接图标](../images/external_link.svg "外部链接图")](https://www.elastic.co/products/kibana "您使用 Elasticsearch 的窗口")，可以查看组件日志。

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
{: #peer-icp-troubleshooting}

<!-- Commenting out per issue #228
### **Problem:** Error when installing helm release by using the helm install command
{: #icp-invalid-helm}

Using the `helm install` command to install the helm release fails with the error:

```
Error: incompatible versions client[v2.9.1] server[v2.7.3+icp]
```

**Solution:**

This error can occur when the helm CLI version on your local machine is
ahead of helm cli version on your server. To resolve the issue, downgrade the
version of your helm CLI by performing the following steps:

1.  Remove current version of helm client from `/usr/local/bin`.  Or you can
optionally just move binary file to a different directory.
```
rm /usr/local/bin/helm
```
{:codeblock}
2.  Follow these instructions to install helm client by using a script that is provided
by [helm](https://docs.helm.sh/using_helm/#from-script), which includes the following commands:

   * `curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh`
   * `chmod 700 get_helm.sh`
   * `./get_helm.sh --version v2.7.2`

   In the last step, ensure the selected version is compatible with ICP 2.1.0.3. `v2.7.2 ` is recommended.

3.  After the helm installation completes, [login](#peer-kubectl-configure}) to your ICP cluster
and run the following command:

 ```
 bx pr cluster-config <cluster_name>
 ```
 {:codeblock}
 sample output:

 ```
 root@xcephu27:~/helmcli# bx pr cluster-config condev2
 Configuring kubectl: /root/.bluemix/plugins/icp/clusters/condev2/kube-config
 Property "clusters.condev2" unset.
 Property "users.condev2-user" unset.
 Property "contexts.condev2-context" unset.
 Cluster "condev2" set.
 User "condev2-user" set.
 Context "condev2-context" created.
 Switched to context "condev2-context".

 Cluster condev2 configured successfully.

 Configuring helm: /root/.helm
 Helm configured successfully

 OK
 ```

4. Add helm home directory from step three to your env:

 ```
 export HELM_HOME=/root/.helm
 ```
 {:codeblock}

5. Check helm version:

  ```
  helm version --tls
  ```
  {:codeblock}

  sample output:

  ```
  root@xcephu27:~/helmcli# helm version --tls
  Client: &version.Version{SemVer:"v2.7.2", GitCommit:"8478fb4fc723885b155c924d1c8c410b7a9444e6", GitTreeState:"clean"}
  Server: &version.Version{SemVer:"v2.7.3+icp", GitCommit:"27442e4cfd324d8f82f935fe0b7b492994d4c289", GitTreeState:"dirty"}
  ```

6. Rerun your `helm install` command.

-->

### **问题：**登录到同级容器时出错
{: #icp-bash-error}

使用 kubectl 工具通过运行以下命令来执行到同级时，发生此问题：

```
kubectl exec -it -n <namespace> <PEER_CONTAINER_NAME> bash
```
{:codeblock}

命令失败并生成类似于以下示例的错误消息：

```
OCI runtime exec failed: exec failed: container_linux.go:348: starting container process caused "open /dev/pts/4294967296: no such file or directory": unknown
```

**解决方案：**

在大尾数法系统（如 IBM System Z）上，在同一个 Docker 容器中打开两个 Docker Bash 会话时，会发生此错误。第二个 Bash 会话可能无法连接到运行中的容器。要解决此问题，请[重新启动同级容器](#peer-restart)，然后在同级重新启动后，重试 `kubectl exec` 命令。
