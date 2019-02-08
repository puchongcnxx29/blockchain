---

copyright:
  years: 2017, 2018
lastupdated: "2018-09-05"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# 在 {{site.data.keyword.cloud_notm}} Private 中操作远程同级
{: #remote-peer-operate}


***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


在设置 {{site.data.keyword.blockchainfull}} Platform Remote Peer on {{site.data.keyword.cloud_notm}} Private (ICP) 后，您需要完成若干操作步骤后，同级才能发出事务来查询和调用区块链网络的分类帐。这些步骤包括将组织添加到通道，将远程同级连接到通道，在远程同级上安装链代码，将通道上的链代码实例化，以及将应用程序连接到远程同级。
{:shortdesc}

您需要从 ICP 集群完成一些必备步骤才能操作远程同级。

**先决条件：**
- [检索远程同级端点信息](#remote-peer-retrieve-endpoint-info)
- [使用 kubectl 登录到同级容器](#remote-peer-kubectl-configure)
- [重新启动远程同级](#remote-peer-restart)

然后，可以使用下列其中一种方法来操作远程同级。

**操作路径：**
- [使用 Fabric SDK](#remote-peer-operate-with-sdk)
- [使用命令行](#remote-peer-cli-operate)

Fabric SDK 是建议的路径，不过指示信息假定您熟悉 SDK 的操作。

建议您至少部署远程同级 Helm 图表的两个实例，以实现[高可用性](/docs/services/blockchain/remote_peer_icp.html#high-availability)。因此，对于每个同级，您需要执行一次以下操作步骤。准备好从应用程序调用和查询链代码时，请连接到这两个同级，以确保[应用程序具有高可用性](/docs/services/blockchain/v10_application.html#ha-app)。

**注**：{{site.data.keyword.blockchainfull_notm}} Platform Remote Peer 无权访问在 {{site.data.keyword.blockchainfull_notm}} Platform 上托管的同级的完整功能，也无权获得完全支持。因此，无法使用“网络监视器”来操作远程同级。开始运行远程同级之前，请确保查看[注意事项](/docs/services/blockchain/remote_peer.html#remote-peer-limitations)。

## 先决条件

无论您是计划使用 SDK 还是命令行，都需要在操作同级过程中完成此部分中的步骤。开始之前，可以完成前两个步骤，以检索同级端点并配置 kubectl。

### 检索远程同级端点信息
{: #remote-peer-retrieve-endpoint-info}

登录到 ICP 控制台。单击左上角的**菜单**图标。单击**工作负载** > **部署**。然后找到并单击远程同级实例的名称以打开部署概述屏幕。向下滚动到**公开详细信息**部分，并找到远程同级的 `Ingress IP` 和`端口`。

```
Ingress IP: <IP address>
Ports: < Port List>
```
{:codeblock}

记下 Ingress IP 地址和 Node 端口以用于连接到远程同级。在远程同级上安装和实例化链代码时，需要使用此信息。在以下示例中，同级地址为 `9.46.126.89:30162`、`9.46.126.89:30260` 或 `9.46.126.89:32051`。

```
Ingress IP: <9.46.126.89>
Ports: <7051 7051/TCP 30162/NodePort7052 7052/TCP 30260/NodePort7053 7053/TCP 32051/NodePort>
```
{:codeblock}

### 配置 kubectl CLI
{: #remote-peer-kubectl-configure}

您需要使用 **kubectl** 命令行工具在 ICP 中运行的同级容器内执行一些必要的操作。使用 {{site.data.keyword.cloud_notm}} Private CLI 工具登录到集群：

```
bx pr login -a https://<cluster_CA_domain>:8443 --skip-ssl-validation

```
{:codeblock}
<!-- removing per defect #208
docker login <cluster_CA_domain>:8500
-->

然后，登录到 ICP 集群的仪表板。单击 ICP UI 右上角的用户头像，然后单击**配置客户机**。在本地计算机的终端窗口中复制、粘贴和运行命令。可以通过运行以下命令来验证配置是否成功生效：

```
kubectl -n <namespace> get pods
```
{:codeblock}

其中，不使用 `default` 名称空间时，应该指定 `-n <namespace>`。将 `<namespace>` 替换为 ICP 集群中名称空间的值。


该命令应该会生成类似于以下示例的响应。`remotepeer-bd65c4769-95rmm` 是同级容器的名称。

```
NAME                                 READY      STATUS             RESTARTS   AGE
remotepeer-bd65c4769-95rmm           2/2        Running            0          6h
```

如果无法识别响应中的远程同级 pod 名称，那么可以在部署面板的 **Pod** 窗格中找到 pod 的名称。

通过运行以下命令，并将 `<PEER_CONTAINER_NAME>` 的值替换为上述 pod 的名称，以执行到远程同级容器。

```
kubectl exec -it -n <namespace> <PEER_CONTAINER_NAME> bash
```
{:codeblock}
其中，不使用 `default` 名称空间时，应该指定 `-n <namespace>`。将 `<namespace>` 替换为 ICP 集群中名称空间的值。

例如：

```
kubectl exec -it remotepeer-bd65c4769-95rmm bash
```
{:codeblock}

您现在已准备就绪，可以使用同级容器 CLI。如果要使用 SDK 或命令行，那么需要使用 CLI 来下载同级 TLS 证书，并将您的签名证书上传到同级。

## 重新启动在 ICP 中运行的远程同级
{: #remote-peer-restart}

每当向远程同级添加证书后，都需要重新启动同级节点，以便该节点识别到新证书。

可以使用 **kubectl** 命令来重新启动在 ICP 中运行的远程同级。该命令需要远程同级 **pod** 及其**容器**的名称作为命令参数。有关使用 kubectl 命令的更多信息，请参阅[配置 kubectl CLI](#remote-peer-kubectl-configure)。

1. 在 ICP 控制台中，找到用于远程同级部署的 **pod** 名称和**容器**名称。

  1. 单击控制台左上角的菜单图标，然后单击**工作负载 > 部署**。
  2. 在表中找到并单击远程同级发布以将其打开。
  3. 向下滚动以查看关联的 **pod** 名称。记下 **pod** 名称。
  4. 单击该 pod 以将其打开。
  5. 单击**容器**选项卡。记下远程同级**容器**名称。

2. 在命令行中，运行以下命令来重新启动同级，并将 `<REMOTE_PEER_POD_NAME>` 和 `<REMOTE_PEER_CONTAINER_NAME>` 替换为上面的值。

  ```
  kubectl exec <REMOTE_PEER_POD_NAME> -c <REMOTE_PEER_CONTAINER_NAME> /sbin/killall5
  ```
  {:codeblock}

3. 可以通过运行 `kubectl get pods` 命令并检查输出中 pod 的 **RESTART** 计数，验证远程同级是否已重新启动。

此外，可以使用 [HEAD 请求](/docs/services/blockchain/monitor_network.html#monitor-nodes)来检查远程同级的可用性。


## 使用 Fabric SDK 来操作远程同级
{: #remote-peer-operate-with-sdk}

Hyperledger Fabric SDK 提供了一组功能强大的 API，支持应用程序与区块链网络进行交互并操作区块链网络。您可以在 [Hyperledger Fabric SDK 社区文档 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/getting_started.html#hyperledger-fabric-sdks "Hyperledger Fabric SDK 社区文档") 中找到 Fabric SDK 中最新的受支持语言列表和完整的可用 API 列表。可以使用 Fabric SDK 将远程同级加入 {{site.data.keyword.blockchainfull_notm}} Platform 上的通道，在同级上安装链代码，然后将通道上的链代码实例化。

以下指示信息使用 [Fabric Node SDK ![外部链接图标](../images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/ "Fabric Node SDK") 来操作远程同级，并假定您先前已经熟悉该 SDK。开始之前，可以使用[开发应用程序教程](../v10_application.html)来学习如何使用 Node SDK，如果已准备好调用和查询链代码，可以将此教程作为使用远程同级开发应用程序的指南。

### 安装 Node SDK

可以使用 NPM 来安装 [Node SDK ![外部链接图标](../images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/ "Node SDK")：
```
npm install fabric-client@1.1
```
{:codeblock}

建议您使用 Node SDK V1.1。

### 准备 SDK 以使用远程同级
{: #remote-peer-node-sdk}

使用 SDK 来操作远程同级之前，需要生成必要的证书（注册），证书用于允许应用程序与 {{site.data.keyword.blockchainfull_notm}} Platform 上的网络以及远程同级进行通信。执行以 **admin** 身份[使用 SDK 注册](/docs/services/blockchain/v10_application.html#enroll-app-sdk)的步骤。[开发应用程序](/docs/services/blockchain/v10_application.html)教程中也将以 **admin** 身份进行注册，因此无需修改样本代码。

### 将签名证书上传到 {{site.data.keyword.blockchainfull_notm}} Platform
{: #remote-peer-upload-sdk}

您需要将 SDK 签名证书上传到 {{site.data.keyword.blockchainfull_notm}} Platform 上的网络，以便应用程序有权操作网络。

- 在 SDK 生成的加密资料所在的目录下名为 admin 的文件中，可以找到签名证书。复制 `certificate` 字段后引号内的证书，证书以 `-----BEGIN CERTIFICATE-----` 开头，并以 `-----END CERTIFICATE-----` 结尾。可以使用 CLI 通过发出 `echo -e"<CERT>" > admin.pem` 命令，将证书转换为 PEM 格式。然后，可以使用“网络监视器”将证书的内容粘贴到区块链网络中。登录到 {{site.data.keyword.blockchainfull_notm}} Platform 上的网络。在“网络监视器”的“成员”屏幕上，单击**证书** > **添加证书**。为证书指定任意名称，然后将内容粘贴到**证书**字段中。当系统询问您是否要重新启动同级时，请单击**重新启动**。必须在组织加入通道之前执行此操作。

### 将签名证书上传到远程同级
{: #remote-peer-upload-signcert}

您还需要将 SDK 的签名证书上传到远程同级并重新启动该远程同级。您需要在远程同级容器内安装已[上传到 {{site.data.keyword.blockchainfull_notm}} Platform](#remote-peer-upload-sdk) 的同一签名证书。

在[同级容器](#remote-peer-kubectl-configure)内运行以下命令以上传证书。
```
cd /mnt/crypto/peer/peer/msp/admincerts/
echo -e "<CERT.PEM>" > cert2.pem
```
{:codeblock}

将文本 `<CERT.PEM>` 替换为您的签名证书，证书以 `-----BEGIN CERTIFICATE-----` 开头，并以 `-----END CERTIFICATE-----` 结尾。请**注意**，此目录中已存在 cert.pem 文件。将新证书命名为 `cert2.pem` 或类似名称，以避免覆盖容器内已存在的证书。

由于添加了新证书，因此需要重新启动容器，以便同级可选取该证书。请遵循以下[指示信息](#remote-peer-restart)来重新启动同级。

### 将远程同级的 TLS 证书传递到 SDK
{: #icp-remote-peer-download-tlscert}

您需要将同级容器中 TLS `cacert.pem` 的内容复制到应用程序，以认证来自 SDK 的通信。在[同级 CLI 容器](#remote-peer-kubectl-configure)中，运行以下命令：
```
cat /mnt/certs/tls/cacert.pem
```
{:codeblock}

将 `cacert.pem` 文件的整个文本复制到剪贴板，该文本以第一个 `-----BEGIN CERTIFICATE-----` 开头，以最后一个 `-----END CERTIFICATE-----` 结尾。以 PEM 格式将此证书保存到本地文件系统。然后，可以使用简单的读取文件命令将该 TLS 证书导入应用程序。
```
var caCert = fs.readFileSync(path.join(__dirname, './cacert.pem'));
```
{:codeblock}

### 向 SDK 提供远程同级端点信息
{: #remote-peer-SDK-endpoints}

找到[远程同级的端点信息](#remote-peer-retrieve-endpoint-info)，然后通过声明新的同级变量或更新连接概要文件，向 SDK 提供这些信息。以下示例将同级定义为 Fabric 网络上的端点，并向其传递导入的 TLS 证书。
```
var peer = fabric_client.newPeer('grpcs://9.46.126.89:31618', { pem:  Buffer.from(caCert).toString(), 'ssl-target-name-override': remotepeer.blockchain.com});
```
{:codeblock}

您需要指定 `<something>.blockchain.com` 的 `ssl-target-name-override`，同级才能解析 DNS 请求。

**注：**由于同级是远程的，因此 {{site.data.keyword.blockchainfull_notm}} Platform 网络上的其他组织将无法在其连接概要文件中找到您的同级端点信息。如果其他组织需要向您的远程同级发送事务以进行背书，您需要在频带内和频带外操作中提供公共 IP 和 TLS 证书。

### 使用 SDK 加入通道
{: #remote-peer-join-channel-sdk}

作为区块链网络的成员，您的组织需要添加到网络中的通道后，才能将远程同级加入通道。

  - 可以为远程同级启动新的通道。作为通道启动者，您可以在[通道创建](/docs/services/blockchain/create_channel.html#creating-a-channel)期间自动包含您的组织。请注意，您需要在 {{site.data.keyword.blockchainfull_notm}} Platform 上至少具有一个同级后，才能在“网络监视器”中创建通道。  

  - 区块链网络的其他成员也可以使用[通道更新](/docs/services/blockchain/create_channel.html#updating-a-channel)将您的组织添加到现有通道。{{site.data.keyword.blockchainfull_notm}} Platform 上同级的通道成员可以使用“网络监视器”将您的组织添加到通道，即使您未在该平台上托管任何同级。

    将组织添加到通道后，需要将同级的签名证书添加到该通道，以便其他成员可以在交易期间验证您的数字签名。在安装期间，远程同级将上传其签名证书，因此您只需要将该证书同步到通道即可。在“网络监视器”的“通道”屏幕中，找到组织加入的通道，然后从**操作**标题下的下拉列表中选择**同步证书**。此操作会同步通道上所有同级的证书。

当您的组织是通道的一部分时，请遵循[加入通道](/docs/services/blockchain/v10_application.html#join-channel-sdk)的指示信息。您需要提供排序服务的 URL 和通道名称。

### 使用 SDK 在同级上安装链代码
{: #remote-peer-install-cc-sdk}

使用以下指示信息通过 SDK 在远程同级上[安装链代码](/docs/services/blockchain/v10_application.html#install-cc-sdk)。

### 使用 SDK 在通道上实例化链代码
{: #remote-peer-instantiate-cc-sdk}

只需要通道中的一个成员对链代码进行实例化或更新。因此，{{site.data.keyword.blockchainfull_notm}} Platform 上同级的通道的任何网络成员都可以使用“网络监视器”来实例化链代码并指定背书策略。但是，如果要使用远程同级在通道上实例化链代码，那么可以使用 SDK 并遵循指示信息来[实例化链代码](/docs/services/blockchain/v10_application.html#instantiate-cc-sdk)。


## 使用 CLI 来操作远程同级
{: #remote-peer-cli-operate}

您还可以通过命令行使用 Fabric CA 客户机和 Fabric Tools 容器来操作远程同级。在以下指示信息中，首先使用 Fabric CA 客户机来生成必需的证书，然后使用 Fabric Tools 容器通过将同级加入通道，安装链代码，然后在通道上实例化链代码来操作同级。

### 使用 Fabric CA 客户机注册
{: #peer-client-enroll}

第一步是使用 Fabric CA 客户机生成必需的证书（注册）。您需要首先安装 Fabric CA 客户机。将[适用于您平台的 fabric-ca 二进制文件 V1.1.0](https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric-ca/hyperledger-fabric-ca/) 下载到本地计算机，对该文件解压缩，然后将其移至文件夹，例如 `$HOME/fabric-ca-remote/`。

1.  准备您的环境以使用 Fabric CA 客户机。切换到将客户机二进制文件移至其中的目录，以便可以直接在命令中对其进行引用。
    ```
    cd $HOME/fabric-ca-remote/
    ```
    {:codeblock}

2.  设置客户机可在其中创建密钥的路径。如果这是首次运行 `enroll` 命令，那么不会存在 `msp` 文件夹和 `.yaml` 文件。如果先前已运行过 `enroll` 命令，请务必使用下面的 `rm` 命令来删除先前注册尝试中的所有配置资料：
    ```
    export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca-remote
    rm -rf $FABRIC_CA_CLIENT_HOME/fabric-ca-client-config.yaml
    rm -rf $FABRIC_CA_CLIENT_HOME/msp
    ```
    {:codeblock}

3.  您需要在 {{site.data.keyword.blockchainfull_notm}} Platform 上检索认证中心的端点信息。在“网络监视器”的“概述”屏幕上，单击**连接概要文件**按钮，然后打开包含网络凭证的 JSON 文件。向下滚动到 `certificateAuthorities` 部分，并记录以下信息。
    ```
    Hostname and port for CA: url（不包含 https 前缀）
    Admin user ID: enrollId
    Admin password: enrollSecret
    CA Name: caName
    ```

4.  您还需要在 {{site.data.keyword.blockchainfull_notm}} 中检索 TLS 证书以用于与 CA 进行通信。以上网络凭证包括必需的证书。在连接概要文件的 `certificateAuthorities` 部分中，复制 `"pem:"` 之后的证书，证书以 `-----BEGIN CERTIFICATE-----` 开头，并以 `-----END CERTIFICATE-----` 结尾。**请勿包含引号**。

    在终端窗口中运行以下命令，并将 `<certificate>` 替换为复制的证书。
    ```
    echo -e "<certificate>" > $HOME/fabric-ca-remote/cert.pem
    ```
    {:codeblock}

 将此证书存储在可在未来命令中引用此证书的位置。

5.  您现在已准备就绪，可以生成必需的证书。运行以下命令：
    ```
    ./fabric-ca-client enroll -u https://<enroll_id>:<enrollSecret>@<ca_hostname_with_port> --caname <ca_name> --tls.certfiles <tls_cert_file>
    ```
    {:codeblock}

    例如：
    ```
    ./fabric-ca-client enroll -u https://admin:C25A062870@ash-zbc07c.4.secure.blockchain.ibm.com:21241 --tls.certfiles $HOME/fabric-ca-remote/cert.pem --caname PeerOrg1CA
    ```
    {:codeblock}

    命令成功完成后，可以看到类似于以下示例的响应：
    ```
    2018/06/13 09:00:45 [INFO] Created a default configuration file at
    /fabric-ca-remote/fabric-ca-client-config.yaml
    2018/06/13 09:00:45 [INFO] TLS Enabled
    2018/06/13 09:00:45 [INFO] generating key: &{A:ecdsa S:256}
    2018/06/13 09:00:45 [INFO] encoded CSR
    2018/06/13 09:00:46 [INFO] Stored client certificate at
    /fabric-ca-remote/msp/signcerts/cert.pem
    2018/06/13 09:00:46 [INFO] Stored root CA certificate
    at /fabric-ca-remote/msp/cacerts/ash-4-secure-blockchain-ibm-com-71139-PeerOrg1CA.pem
    2018/06/13 09:00:46 [INFO] Stored intermediate CA certificates at
    /fabric-ca-remote/intermediatecerts/ash-4-secure-blockchain-ibm-com-71139-PeerOrg1CA.pem
    ```

    该命令将生成必需的证书，并将这些证书存储在 `$FABRIC_CA_CLIENT_HOME` 中名为 `msp` 的文件夹中。此文件夹及其包含的证书在以下步骤中很重要。

### 管理本地系统上的证书
{: #manage-certs}

要能够操作远程同级，需要先对本地计算机上的证书执行一些管理操作，并将 Fabric CA 客户机生成的某些证书上传到 {{site.data.keyword.blockchainfull_notm}} Platform 和同级。此外，还需要从 Platform 和同级下载 TLS 证书。如果要了解有关将使用的证书以及要执行的任务的更多信息，请参阅[在 {{site.data.keyword.blockchainfull_notm}} Platform 上管理证书](/docs/services/blockchain/certificates.html)。

在本地计算机上，打开命令终端并浏览至将 Fabric-CA-Client 二进制文件移至其中并存储了 MSP 文件夹的目录。
1. 将 `cert.pem` 文件从 `signcerts` 文件夹复制到新的 `admincerts` 文件夹。  
  ```
  cd $FABRIC_CA_CLIENT_HOME/msp
  mkdir admincerts
  cd admincerts/
  cp ../signcerts/cert.pem  .
  ```
  {:codeblock}

2. 您还需要将远程同级的 TLS 证书从 ICP 上的同级容器复制到本地计算机上。

  - [登录到同级容器](#remote-peer-kubectl-configure)并运行以下命令：
    ```
    cat /mnt/certs/tls/cacert.pem
    ```
    {:codeblock}

    将 `cacert.pem` 文件的整个文本复制到剪贴板，该文本以第一个 `-----BEGIN CERTIFICATE-----` 开头，以最后一个 `-----END CERTIFICATE-----` 结尾。

  - 切换回本地计算机并运行以下命令。将文本 `<CACERT.PEM>` 替换为剪贴板中的文本。
    ```
    cd /$FABRIC_CA_CLIENT_HOME/msp
    mkdir tls
    cd tls/
    echo -e "<CACERT.PEM>" > cacert.pem
    ```
    {:codeblock}

3. 要授予 CLI 在同级上安装链代码的权限，您需要将 Fabric CA 客户机生成的签名证书上传到同级的管理文件夹，然后重新启动同级。因此，请将本地计算机上的 `admincert/cert.pem` 证书复制到同级容器的 `/mnt/crypto/peer/peer/msp/admincerts/` 目录中作为 `cert2.pem`。

    - 在本地计算机上，运行以下命令：

      ```
      cd $FABRIC_CA_CLIENT_HOME/msp/admincerts
      cat cert.pem
      ```
      {:codeblock}

    - 将 `cert.pem` 文件的文本复制到剪贴板，该文本以 `-----BEGIN CERTIFICATE-----` 开头，以 `-----END CERTIFICATE-----` 结尾。

    - 切换回同级容器，并运行以下命令将 cert.pem 添加到同级证书。将文本 `<CERT.PEM>` 替换为剪贴板中的文本。

      ```
      cd /mnt/crypto/peer/peer/msp/admincerts/
      echo -e "<CERT.PEM>" > cert2.pem
      ```
      {:codeblock}

      **注：**此目录中已存在 cert.pem 文件。请勿覆盖该文件。

    - 由于添加了新证书，因此需要重新启动容器，以便同级可选取该证书。请遵循以下[指示信息](#remote-peer-restart)来重新启动同级。

4. 您需要将相同的 `admincert/cert.pem` 证书从本地计算机上传到 {{site.data.keyword.blockchainfull_notm}} Platform，以便远程 CLI 或应用程序执行通道操作，例如访存通道起源区块和实例化链代码。
    1. 在本地计算机上，对文件`/$FABRIC_CA_CLIENT_HOME/mssp/admincerts/cert.pem` 执行 cat 命令，然后将其复制到剪贴板。
    2. 进入 {{site.data.keyword.blockchainfull_notm}} Platform 上网络的“网络监视器”。
    3. 单击左侧导航器中的**成员**，然后单击**证书**选项卡。
    4. 单击**添加证书**按钮。
    5. 在**添加证书**窗口中，为证书提供名称（如 `fabrictools.pem`），粘贴刚才复制到剪贴板的证书，然后单击**提交**。
    6. 系统将询问您是否要重新启动同级。单击**重新启动**。
    7. 单击左侧导航器中的**通道**，并找到远程同级将加入的通道。
    8. 单击**操作**标题下的三个点，然后单击**同步证书**。在**同步证书**窗口中，单击**提交**。

    **注**：在同级加入下面的通道之前，务必同步通道证书。

### 设置 Fabric Tools 容器

将所有证书移至必需的位置后，即可随时安装并使用 Docker 的 Fabric Tools 容器。同样，需要从您存储 MSP 文件夹的目录运行这些命令。

1.  使用以下命令来下载 Fabric Tools Docker 映像：
    ```
    docker pull ibmblockchain/fabric-tools:1.1.0
    ```
    {:codeblock}

2.  运行以下命令来启动 Fabric Tools 容器。
    ```
    docker network create blockchain.com
    docker run -ti --network blockchain.com -v ${PWD}:/mnt ibmblockchain/fabric-tools:1.1.0
    ```
    {:codeblock}

    此命令会在 Tools 容器内安装 MSP 目录。

3.  远程同级地址必须解析为具有 `blockchain.com` 后缀的域名。这可通过 DNS 或修改 /etc/hosts 文件来实现。在以下指示信息中，将修改 Fabric Tools 容器中的 `/etc/hosts` 文件。请登录到 ICP 控制台，找到远程同级[端点信息](#remote-peer-retrieve-endpoint-info)。具有同级 `INGRESS_IP` 后，请使用远程同级的地址来运行以下命令。
    ```
    echo -e "<INGRESS_IP> remotepeer.blockchain.com" >> /etc/hosts
    ```
    {:codeblock}

### 使用 Fabric Tools 容器将远程同级加入通道
{: #icp-cli-join-peer-to-channel}

需要将组织添加到网络中的通道后，才能运行 CLI 命令将远程同级加入通道。

  - 可以为远程同级启动新的通道。作为通道启动者，您可以在[通道创建](/docs/services/blockchain/create_channel.html#creating-a-channel)期间自动包含您的组织。请注意，您需要在 {{site.data.keyword.blockchainfull_notm}} Platform 上至少具有一个同级后，才能在“网络监视器”中创建通道。  

  - 区块链网络的其他成员也可以使用[通道更新](/docs/services/blockchain/create_channel.html#updating-a-channel)将您的组织添加到现有通道。{{site.data.keyword.blockchainfull_notm}} Platform 上同级的通道成员可以使用“网络监视器”将您的组织添加到通道，即使您未在该平台上托管任何同级。

    将组织添加到通道后，需要将同级的签名证书添加到该通道，以便其他成员可以在交易期间验证您的数字签名。在安装期间，远程同级将上传其签名证书，因此您只需要将该证书同步到通道即可。在“网络监视器”的“通道”屏幕中，找到组织加入的通道，然后从**操作**标题下的下拉列表中选择**同步证书**。此操作会同步通道上所有同级的证书。

1. 从“网络监视器”的**概述**面板中提供的 `creds.json` 文件中检索配置信息。单击**连接概要文件**，然后单击**下载**。

  -  通过搜索`排序者 > URL` 下的**排序者**来查找排序者 URL。记下以网络名称开头的 URL。该 URL 类似于以下示例：
    ```
    ash-zbc07b.4.secure.blockchain.ibm.com:21239
    ```

  - 通过搜索**组织**来查找组织的名称。这应该是您用于注册远程同级的组织。您可以找到组织的名称及其关联的 `mspid`。请记下 `mspid` 的值。

2. [查找远程同级端点信息](#remote-peer-retrieve-endpoint-info)，这次记下 `Ingress IP` 旁边列出的`端口`。

  ```
  Ingress IP: <IP address>
  Ports: < Port List>
  ```
  {:codeblock}

  记下 30000 范围内的端口。这些是 Fabric Tools 容器中的可寻址端口，将使用这些端口来设置 `PEERADDR` 环境变量。

  ```
  Ingress IP: <9.46.126.89>
  Ports: <7051 7051/TCP 30162/NodePort7052 7052/TCP 30260/NodePort7053 7053/TCP 32051/NodePort>
  ```
  {:codeblock}

  因此，使用的 `PEERADDR` 是通过在 `/etc/hosts` 文件中指定的值以及绑定到 `7051` 的端口（即，示例中的 `30162`）一起构造的。所以，生成的 `PEERADDR` 值将为 `remotepeer.blockchain.com:30162`。

3. 运行以下命令在 Fabric Tools 容器中设置环境变量。

  ```
  export ORDERER_1=<ORDERER_URL>
  export CHANNEL=<CHANNEL_NAME>
  export CC_NAME=<CC_NAME>
  export ORGID=<ORGANIZATION_MSP_ID>
  export PEERADDR=<PEER_ADDR>
  ```
  {:codeblock}

  将相关字段替换为您自己的信息。
    - 将 `<ORDERER_URL>` 替换为 `creds.json` 文件中排序者的主机名和端口。
    - 将 `<CHANNEL_NAME>` 替换为同级加入的通道的名称。
    - 将 `<CC_NAME>` 替换为要引用链代码的任何名称。
    - 将 `<ORGANIZATION_MSP_ID>` 替换为 `creds.json` 文件中组织的名称。
    - 将 `<PEER_ADDR>` 替换为在上一步中构造的值。

  例如：

  ```
  export ORDERER_1=ash-zbc07b.4.secure.blockchain.ibm.com:21239
  export CHANNEL=defaultchannel
  export CC_NAME=mycc
  export ORGID=PeerOrg1
  export PEERADDR=remotepeer.blockchain.com:30162
  ```
  {:codeblock}

4. 访存通道的起源区块以在同级上构建通道分类帐。首先使用 `cd` 切换到根目录，然后运行以下命令来访存起源区块。

  ```
  cd /

  CORE_PEER_TLS_ROOTCERT_FILE=/mnt/msp/tls/cacert.pem CORE_PEER_TLS_ENABLED=true CORE_PEER_ADDRESS=${PEERADDR} CORE_PEER_LOCALMSPID=${ORGID} CORE_PEER_MSPCONFIGPATH=/mnt/msp/ GOPATH=/ peer channel fetch 0 -o ${ORDERER_1} -c ${CHANNEL} --cafile /mnt/msp/tls/cacert.pem --tls
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

5. 使用以下命令将同级加入通道：

  ```
  CORE_PEER_TLS_ROOTCERT_FILE=/mnt/msp/tls/cacert.pem CORE_PEER_TLS_ENABLED=true CORE_PEER_ADDRESS=${PEERADDR} CORE_PEER_LOCALMSPID=${ORGID} CORE_PEER_MSPCONFIGPATH=/mnt/msp/ GOPATH=/ peer channel join -b ${CHANNEL}_0.block
  ```
  {:codeblock}

  此命令成功执行后，应该会看到类似于以下示例的信息：

  ```
  2018-07-06 18:32:09.509 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
  2018-07-06 18:32:09.992 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
  2018-07-06 18:32:09.992 UTC [main] main -> INFO 003 Exiting.....
  ```

### 使用 Fabric Tools 容器在远程同级上安装链代码
{: #icp-toolcontainer-install-cc}
现在，我们已准备就绪，可以在远程同级上安装和实例化链代码。对于这些指示信息，将安装 `fabric-samples` 存储库中的 `fabcar` 链代码。使用以下命令从 `github.com` 下载 `fabric-samples` 链代码：

  ```
  cd /
  mkdir /src
  cd /src
  git clone https://github.com/hyperledger/fabric-samples
  cd /
  ```
  {:codeblock}

运行以下同级 CLI 命令将 `fabcar` 链代码安装到远程同级上。

  ```
  GOPATH=/ CORE_PEER_TLS_ROOTCERT_FILE=/mnt/msp/tls/cacert.pem CORE_PEER_TLS_ENABLED=true CORE_PEER_ADDRESS=${PEERADDR} CORE_PEER_LOCALMSPID=${ORGID} CORE_PEER_MSPCONFIGPATH=/mnt/msp/ GOPATH=/ peer chaincode install -n ${CC_NAME} -v v0 -p fabric-samples/chaincode/fabcar/go/
  ```
  {:codeblock}

  此命令成功完成后，应该会看到类似于下面的内容：

  ```
  2018-07-06 18:39:00.461 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
  2018-07-06 18:39:00.461 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
  2018-07-06 18:39:01.142 UTC [main] main -> INFO 003 Exiting.....
  ```

### 使用 Fabric Tools 容器在通道上实例化链代码
{: #icp-toolcontainer-instantiate-cc}

因为只需要一个同级在通道上实例化链代码，所以不必使用远程同级来实例化链代码。在 {{site.data.keyword.blockchainfull_notm}} Platform 上托管的同级可以执行此操作。但是，如果您希望远程同级将链代码实例化到通道，那么可以在 Fabric Tools 容器中运行以下命令：

```
CORE_PEER_TLS_ROOTCERT_FILE=/mnt/msp/tls/cacert.pem CORE_PEER_TLS_ENABLED=true CORE_PEER_ADDRESS=${PEERADDR} CORE_PEER_LOCALMSPID=${ORGID} CORE_PEER_MSPCONFIGPATH=/mnt/msp/ GOPATH=/ peer chaincode instantiate -o ${ORDERER_1} -C ${CHANNEL} -n ${CC_NAME} -v v0 -c '{"Args":[""]}' --tls --cafile /mnt/msp/tls/cacert.pem -P ""
```
{:codeblock}

此命令成功完成后，应该会看到类似于下面的内容：

```
2018-07-06 18:43:15.066 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-07-06 18:43:15.066 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-07-06 18:43:50.227 UTC [main] main -> INFO 003 Exiting.....
```


## 在 {{site.data.keyword.cloud_notm}} Private 中查看远程同级日志
{: #remote-peer-log-icp}

您可以通过 ICP 控制台访问远程同级日志，并在 Kibana 界面中检查日志。

1. 在 ICP 控制台中，单击左上角的**菜单**图标。
2. 从菜单列表中，单击**工作负载** > **Helm 发布**。
3. 在“Helm 发布”表中，单击您的 Helm 发布的名称。
4. 向下滚动到 **Pod** 部分，然后单击表行末尾的**查看日志**链接。这将在 Kibana 界面中打开远程同级日志。

## 更新链代码

一段时间后，您可能需要修改在远程同级上运行的链代码。由于链代码是安装在同级上并在通道上实例化的，因此需要更新通道上所有同级上的链代码。

完成以下步骤以更新链代码：

1. 要在每个远程同级上更新链代码，只需使用客户机应用程序或 CLI 命令重新运行用于在同级上安装链代码的进程即可。请确保指定的链代码名称与最初使用的链代码名称相同。但是，这次将链代码`版本`递增。所有同级都需要使用相同的链代码名称和版本。

2. 在通道上的所有同级上安装新的链代码后，使用“网络监视器”或[同级链代码升级 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/peerchaincode.html#peer-chaincode-upgrade) 命令来更新通道，以使用新的链代码。

有关使用“网络监视器”的“安装代码”面板更新通道上的链代码的更多信息，请参阅这些[指示信息](/docs/services/blockchain/install_instantiate_chaincode.html#updating-a-chaincode)中的步骤 2。

## 故障诊断
{: #icp-troubleshooting}

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

2.  Follow these instructions to install helm client by using a script that is provided
by [helm](https://docs.helm.sh/using_helm/#from-script), which includes the following commands:

   * `curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh`
   * `chmod 700 get_helm.sh`
   * `./get_helm.sh --version v2.7.2`

   In the last step, ensure the version selected is compatible with ICP 2.1.0.3, `v2.7.2 ` is recommended.

3.  After the helm install completes, [login](#remote-peer-kubectl-configure}) to your ICP cluster
and run the following command:

 ```
 bx pr cluster-config <cluster_name>
 ```

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

5. Check helm version:

  ```
  helm version --tls
  ```

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
命令失败并生成类似于以下示例的错误消息：
```
OCI runtime exec failed: exec failed: container_linux.go:348: starting container process caused "open /dev/pts/4294967296: no such file or directory": unknown
```

**解决方案：**  

在大尾数法系统（如 IBM System Z）上，在同一个 Docker 容器中打开两个 Docker Bash 会话时，会发生此错误。第二个 Bash 会话可能无法连接到运行中的容器。要解决此问题，请[重新启动同级容器](#remote-peer-restart)，然后在同级重新启动后，重试 `kubectl exec` 命令。
