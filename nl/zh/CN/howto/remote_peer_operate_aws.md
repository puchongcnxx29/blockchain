---

copyright:
  years: 2017, 2018
lastupdated: "2018-11-27"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# 在 AWS 中操作同级
{: #remote-peer-operate-aws}


***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


在 AWS 中设置 {{site.data.keyword.blockchainfull}} Platform 同级之后，您需要完成若干操作步骤后，您的同级才可以发出事务来查询和调用区块链网络的分类帐。这些步骤包括将组织添加到通道，将同级连接到通道，在同级上安装链代码，在通道上实例化链代码，以及将应用程序连接到同级。
您可以使用 [Fabric SDK](#aws-peer-operate-with-sdk) 或[命令行](#aws-peer-cli-operate)来完成这些操作步骤。Fabric SDK 是建议的路径，不过指示信息假定您熟悉 SDK 的操作。

**注**：AWS 上的 {{site.data.keyword.blockchainfull_notm}} Platform 同级无权访问在 {{site.data.keyword.blockchainfull_notm}} Platform 上托管的同级的完整功能，也无权获得完全支持。因此，无法使用“网络监视器”来操作同级。在 AWS 中开始运行同级之前，请确保阅读[注意事项](/docs/services/blockchain/remote_peer.html#remote-peer-limitations)。

## 使用 Fabric SDK 来操作同级
{: #aws-peer-operate-with-sdk}

Hyperledger Fabric SDK 提供了一组功能强大的 API，支持应用程序与区块链网络进行交互并操作区块链网络。您可以在 [Hyperledger Fabric SDK 社区文档 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/getting_started.html#hyperledger-fabric-sdks "Hyperledger Fabric SDK 社区文档") 中找到 Hyperledger Fabric SDK 中最新的受支持语言列表和完整的可用 API 列表。可以使用 Fabric SDK 将同级加入 {{site.data.keyword.blockchainfull_notm}} Platform 上的通道，在同级上安装链代码，然后在通道上实例化链代码。

以下指示信息使用 [Fabric Node SDK ![外部链接图标](../images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/ "Hyperledger Fabric SDK for jode.js") 来操作同级，并假定您先前已经熟悉 SDK。开始之前，可以使用[开发应用程序教程](/docs/services/blockchain/v10_application.html)来学习如何使用 Node SDK，如果已准备好调用和查询链代码，可以将此教程作为使用同级开发应用程序的指南。

AWS 上的 {{site.data.keyword.blockchainfull_notm}} Platform 同级快速入门创建了两个同级，实现了高可用性。因此，对于每个同级，您需要执行一次操作步骤。准备好从应用程序查询和调用链代码时，请将 SDK 连接到这两个同级，以确保[应用程序具有高可用性](/docs/services/blockchain/v10_application.html#ha-app)。

### 安装 Node SDK

可以使用 NPM 来安装 [Node SDK ![外部链接图标](../images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/ "Hyperledger Fabric SDK for jode.js")：
```
npm install fabric-client@1.2
```
{:codeblock}

建议您使用 Node SDK V1.2。

### 准备 SDK 以使用同级
{: #remote-peer-node-sdk}

使用 SDK 来操作同级之前，需要生成必要的证书（注册），证书用于允许应用程序与 {{site.data.keyword.blockchainfull_notm}} Platform 上的网络以及同级进行通信。执行以 **admin** 身份[使用 SDK 注册](/docs/services/blockchain/v10_application.html#enroll-app-sdk)的步骤。[开发应用程序](../v10_application.html)教程中也将以 **admin** 身份进行注册，因此无需修改样本代码。

### 将 signCert 上传到 IBM Blockchain Platform
{: #remote-peer-upload-SDK}

您需要将 SDK 签名证书上传到 {{site.data.keyword.blockchainfull_notm}} Platform 上的网络，以便其他成员可以识别您的数字签名。

- 在 SDK 生成的加密资料所在的目录下名为 admin 的文件中，可以找到签名证书。复制 `certificate` 字段后引号内的证书，证书以 `-----BEGIN CERTIFICATE-----` 开头，并以 `-----END CERTIFICATE-----` 结尾。可以使用 CLI 通过发出 `echo -e"<CERT>" > admin.pem` 命令，将证书转换为 PEM 格式。然后，可以使用“网络监视器”将证书的内容粘贴到区块链网络中。登录到 {{site.data.keyword.blockchainfull_notm}} Platform 上的网络。在“网络监视器”的“成员”屏幕上，单击**证书** > **添加证书**。为证书指定任意名称，然后将内容粘贴到**证书**字段中。当系统询问您是否要重新启动同级时，请单击**重新启动**。必须在组织加入通道之前执行此操作。

### 将 signCert 上传到同级
{: #remote-peer-upload-signcert}

您还需要将 SDK 的签名证书上传到远程同级并重新启动该远程同级。您需要在远程同级容器中安装[已上传到 IBM Blockchain Platform](#remote-peer-upload-SDK) 的同一签名证书。

在 AWS 控制台中选择实例（单击**服务 > EC2 > 实例**），然后单击“连接”按钮，以通过 SSH 登录到 VPC 实例。遵循 AWS 中的指示信息发出 ssh 命令。

在同级容器中运行以下命令以上传证书。  
```
cd /etc/hyperledger/<PEER_ENROLL_ID>/msp/admincerts/
echo -e "<CERT.PEM>" > cert2.pem
```
{:codeblock}

- 将 `<PEER_ENROLL_ID>` 替换为快速入门模板中指定的并且与此远程同级实例关联的登记标识。  
- 将 `<CERT.PEM>` 替换为您的 signCert，证书以 `-----BEGIN CERTIFICATE-----` 开头，并以 `-----END CERTIFICATE-----` 结尾。    

  **注：**如果文件 `cert.pem` 存在，不要将其覆盖，而是创建一个新文件，例如，`cert2.pem`。

由于添加了新证书，因此需要重新启动容器，以便同级可选取该证书。请遵循以下[指示信息](#remote-peer-aws-restart)来重新启动同级。

### 将同级的 TLS 证书传递到 SDK
{: #aws-remote-peer-download-tlscert}

您需要将同级容器中 TLS `cacert.pem` 的内容复制到应用程序，以认证来自 SDK 的通信。在同级 CLI 容器中运行以下命令。将 `<PEER_ENROLL_ID>` 替换为快速启动模板中指定的远程同级的堆栈名称。（回想一下创建了两个 VPC 实例）。
```
cat /etc/hyperledger/<PEER_ENROLL_ID>/tls/ca.crt
```
{:codeblock}

将 `ca.crt` 文件的整个文本复制到剪贴板，该文本以第一个 `-----BEGIN CERTIFICATE-----` 开头，以最后一个 `-----END CERTIFICATE-----` 结尾。以 PEM 格式将此证书保存到本地文件系统。然后，可以使用简单的读取文件命令将该 TLS 证书导入应用程序。
```
var caCert = fs.readFileSync(path.join(__dirname, './ca.pem'));
```
{:codeblock}

### 向 SDK 提供同级端点信息
{: #remote-peer-SDK-endpoints}

要检索同级的端点信息，请在 AWS 控制台中找到同级实例。记下 EC2 实例的 `AWS EC2 dashboard Public DNS (IPv4)` 值，并通过声明新的同级变量或者更新连接概要文件将其提供给 SDK。将公共 DNS 地址与端口
`7051` 并置。以下示例定义 Fabric 网络上的同级，并向其传递您导入的 TLS 证书。

```
var peer = fabric_client.newPeer('grpcs://<AWS_EC2_dashboard_Public_DNS>:7051', { pem:  Buffer.from(caCert).toString()});
```
{:codeblock}

**注：**由于同级是远程的，因此 {{site.data.keyword.blockchainfull_notm}} Platform 网络上的其他组织将无法在其连接概要文件中找到您的同级的端点信息。如果其他组织需要向您的同级发送要背书的事务，您需要在频带内和频带外操作中提供公共 IP 和 TLS 证书。

### 使用 SDK 加入通道
{: #remote-peer-join-channel-sdk}

作为区块链网络的成员，您的组织需要添加到网络中的通道后，才能将同级加入通道。

  - 可以为同级启动新的通道。作为通道启动者，您可以在[通道创建](/docs/services/blockchain/create_channel.html#creating-a-channel)期间自动包含您的组织。

  - 区块链网络的其他成员也可以使用[通道更新](/docs/services/blockchain/create_channel.html#updating-a-channel)将您的组织添加到现有通道。

    将组织添加到通道后，需要将同级的签名证书添加到该通道，以便其他成员可以在交易期间验证您的数字签名。在安装期间，同级将上传其签名证书，因此您只需要将该证书同步到通道即可。在“网络监视器”的“通道”屏幕中，找到组织加入的通道，然后从**操作**标题下的下拉列表中选择**同步证书**。此操作会同步通道上所有同级的证书。您可能需要等待几分钟，以便可以在发出连接通道命令之前完成通道同步。

在您的组织是通道的一部分后，请遵循[加入通道](/docs/services/blockchain/v10_application.html#join-channel-sdk)的指示信息。您需要提供排序服务的 URL 和通道名称。

### 使用 SDK 在同级上安装链代码
{: #remote-peer-install-cc-sdk}

遵循指示信息使用 SDK 在同级上[安装链代码](/docs/services/blockchain/v10_application.html#install-cc-sdk)。

### 使用 SDK 在通道上实例化链代码
{: #remote-peer-instantiate-cc-sdk}

只需要通道中的一个成员对链代码进行实例化或更新。因此，{{site.data.keyword.blockchainfull_notm}} Platform 上同级的通道的任何网络成员都可以使用“网络监视器”来实例化链代码并指定背书策略。但是，如果要使用同级在通道上实例化链代码，那么可以使用 SDK 并遵循指示信息来[实例化链代码](/docs/services/blockchain/v10_application.html#instantiate-cc-sdk)。


## 使用 CLI 来操作同级
{: #aws-peer-cli-operate}

您还可以通过命令行使用 Fabric CA 客户机和 Fabric Tools 容器来操作同级。在这些指示信息中，我们将首先使用 Fabric CA 客户机来生成必需的证书。然后使用 Fabric Tools 容器通过将同级加入通道，安装链代码，然后在通道上实例化链代码来操作同级。

### 使用 Fabric CA 客户机注册
{: #peer-client-enroll}

第一步是使用 Fabric CA 客户机生成必需的证书（注册）。您需要首先安装 Fabric CA 客户机。将[适用于您平台的 fabric-ca 二进制文件 V1.2.0](https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric-ca/hyperledger-fabric-ca/) 下载到本地计算机，对该文件解压缩，然后将其移至文件夹，例如 `$HOME/fabric-ca-remote/`。

1.  准备您的环境以使用 Fabric CA 客户机。切换到将客户机二进制文件移至的目录，从而可直接在命令中进行引用。
    ```
    cd $HOME/fabric-ca-remote/
    ```
    {:codeblock}

2.  设置客户机可在其中创建密钥的路径。如果之前未运行 `enroll` 命令，那么 `msp` 文件夹和 `.yaml` 文件不存在。
    如果先前已运行过 `enroll` 命令，请务必使用下面的 `rm` 命令来删除先前注册尝试中的所有配置资料：
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

4.  您还需要在 {{site.data.keyword.blockchainfull_notm}} Platform 中检索 TLS 证书以用于与 CA 进行通信。以上网络凭证包括必需的证书。在连接概要文件的 `certificateAuthorities` 部分中，复制 `"pem:"` 之后的证书，证书以 `-----BEGIN CERTIFICATE-----` 开头，并以 `-----END CERTIFICATE-----` 结尾。**请勿包含引号**。

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

要能够操作同级，需要先对本地计算机上的证书执行一些管理操作，并将 Fabric CA 客户机生成的某些证书上传到 {{site.data.keyword.blockchainfull_notm}} Platform 和同级。此外，还需要从 Platform 和同级下载 TLS 证书。如果要了解有关将使用的证书以及将执行的任务的更多信息，请访问[在 {{site.data.keyword.blockchainfull_notm}} Platform 上管理证书](/docs/services/blockchain/certificates.html)。

在本地计算机上，打开命令终端并浏览至将 Fabric-CA-Client 二进制文件移至其中并存储了 MSP 文件夹的目录。

1. 将 `cert.pem` 文件从 `signcerts` 文件夹复制到新的 `admincerts` 文件夹。
   ```
   cd /$FABRIC_CA_CLIENT_HOME/msp/
   mkdir admincerts
   cd admincerts/
   cp ../signcerts/cert.pem  .
   ```
   {:codeblock}

2. 将您的排序节点 TLS 证书从 {{site.data.keyword.blockchainfull_notm}} Platform 复制到 MSP 文件夹。您的连接概要文件包含必要的证书。在“网络监视器”的“概述”屏幕上，单击**连接概要文件**按钮，然后打开包含网络凭证的 JSON 文件。导航到`排序节点`部分。复制 `"pem:"` 后的证书，证书以 `-----BEGIN CERTIFICATE-----` 开头，并以 `-----END CERTIFICATE----- ` 结尾。不要包含引号。

    从终端窗口，运行以下命令：
    ```
    cd /$FABRIC_CA_CLIENT_HOME/msp
    echo -e "<paste in the certificate here>" > orderer_tlscacert.pem
    ```
    {:codeblock}

3. 您还需要将同级的 TLS 证书从 AWS 上的同级容器复制到本地计算机上。

    - [遵循这些指示信息](/docs/services/blockchain/remote_peer_aws.html#remote-peer-aws-test)登录到同级容器并运行以下命令，将“<PEER_ENROLL_ID>”替换为“快速启动”模板中指定的同级的堆栈名称（随附编号）。（回想一下创建了两个 VPC 实例。）
      ```
      cat /etc/hyperledger/<PEER_ENROLL_ID>/tls/ca.crt
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

      **注：**缺省情况下，AWS 快速启动模板在两个不同可用性区域中创建两个同级实例。如果已针对其中一个同级完成这些步骤，那么当您对第二个实例运行这些步骤时，cacert.pem 文件将已存在。请继续，并将其替换为来自第二个同级的证书。

4. 要授予 CLI 在同级上安装链代码的权限，请将 Fabric CA 客户机生成的 signCert 上传到同级的管理员文件夹，然后重新启动该同级。因此，请将本地计算机上的
`admincert/cert.pem` 证书作为 `cert2.pem` 复制到同级容器的 `/etc/hyperledger/<PEER_ENROLL_ID>/msp/admincerts/` 目录下。

    - 在本地计算机上，运行以下命令：

      ```
      cd /$FABRIC_CA_CLIENT_HOME/msp/admincerts
      cat cert.pem
      ```
      {:codeblock}

    - 将 `cert.pem` 文件的文本复制到剪贴板，该文本以 `-----BEGIN CERTIFICATE-----` 开头，以 `-----END CERTIFICATE-----` 结尾。

    - 切换回同级容器，并运行以下命令将 cert.pem 添加到同级证书。将文本 `<CERT.PEM>` 替换为剪贴板中的文本。

      ```
      cd /etc/hyperledger/<PEER_ENROLL_ID>/msp/admincerts/
      echo -e "<CERT.PEM>" > cert2.pem
      ```
      {:codeblock}

      **注：**此目录中已存在 cert.pem 文件。请勿覆盖该文件。

    - 由于添加了新证书，我们需要重新启动容器，以便同级可选取该证书。请遵循以下指示信息[重新启动同级容器](#remote-peer-aws-restart)。

5. 将相同的 `admincert/cert.pem` 证书从本地计算机上传到 {{site.data.keyword.blockchainfull_notm}} Platform，以便远程 CLI 或应用程序执行通道操作，例如访存通道起源区块和实例化链代码。
    1. 在本地计算机上，对文件`/$FABRIC_CA_CLIENT_HOME/mssp/admincerts/cert.pem` 执行 cat 命令，然后将其复制到剪贴板。
    2. 进入 {{site.data.keyword.blockchainfull_notm}} Platform 上网络的“网络监视器”。
    3. 单击左侧导航器中的**成员**，然后单击**证书**选项卡。
    4. 单击**添加证书**按钮。
    5. 在**添加证书**窗口中，为证书提供名称（如 `fabrictools.pem`），粘贴刚才复制到剪贴板的证书，然后单击**提交**。
    6. 系统将询问您是否要重新启动同级。单击**重新启动**。
    7. 单击左侧导航器中的**通道**，并找到同级将加入的通道。
    8. 单击**操作**标题下的三个点，然后单击**同步证书**。在**同步证书**窗口中，单击**提交**。

    **注**：在同级加入通道或在通道上实例化链代码之前，务必同步通道证书。您可能需要等待几分钟，以便可以在发出加入通道或初始化链代码的命令之前完成通道同步。   

### 设置 Fabric Tools 容器
{: #setup-fabric-cli}

将所有证书移至必需的位置后，即可随时安装并使用 Docker 的 Fabric Tools 容器。确保从您存储 MSP 文件夹的目录运行这些命令。

1.  使用以下命令来下载 Fabric Tools Docker 映像：
    ```
    docker pull ibmblockchain/fabric-tools:1.2.0
    ```
    {:codeblock}

2.  运行以下命令来启动 Fabric Tools 容器。
    ```
    docker network create blockchain.com
    docker run -ti --network blockchain.com -v ${PWD}:/mnt ibmblockchain/fabric-tools:1.2.0
    ```
    {:codeblock}

    此命令会在 Tools 容器内安装 MSP 目录。

<!--
3.  The peer address must resolve to a domain name with a suffix of `blockchain.com`. You can do this either via your DNS or modify your /etc/hosts file. For purposes of these instructions, we will modify the `/etc/hosts` file in the Fabric tools container. To retrieve the endpoint information of your peer, locate your peer instance in the AWS console. Make note of the value of the AWS `IPv4 Public IP` for the EC2 instance.

    ```
    echo -e "<AWS_IPv4_PUBLIC_IP> remotepeer.blockchain.com" >> /etc/hosts
    ```
    {:codeblock}
-->

### 使用 Fabric Tools CLI 将同级加入通道
{: #fabric-cli-join-peer-to-channel}

需要将组织添加到网络中的通道后，才能运行 CLI 命令将同级加入通道。

  - 可以为同级启动新的通道。作为通道启动者，您可以在[通道创建](/docs/services/blockchain/create_channel.html#creating-a-channel)期间自动包含您的组织。

  - 区块链网络的其他成员也可以使用[通道更新](/docs/services/blockchain/create_channel.html#updating-a-channel)将您的组织添加到现有通道。

    将组织添加到通道后，需要将同级的签名证书添加到该通道，以便其他成员可以在交易期间验证您的数字签名。在安装期间，同级将上传其签名证书，因此您只需要将该证书同步到通道即可。在“网络监视器”的“通道”屏幕中，找到组织加入的通道，然后从**操作**标题下的下拉列表中选择**同步证书**。此操作会同步通道上所有同级的证书。

1. 从“网络监视器”的“概述”面板中提供的`连接概要文件`检索配置信息。单击**连接概要文件**，然后单击**下载**。

  -  通过搜索`排序者 > URL` 下的**排序者**来查找排序者 URL。记下以网络名称开头的 URL。该 URL 类似于以下示例：
    

    ```
    ash-zbc07b.4.secure.blockchain.ibm.com:21239
    ```

  - 通过搜索**组织**来查找组织的名称。这应该是您用于注册同级的组织。您可以找到组织的名称及其关联的 `mspid`。请记下 `mspid` 的值。

2. 在 AWS EC2 仪表板公用 DNS (IPv4) 中找到 AWS VPC 同级（公用 DNS IPv4 地址）的端点信息。此 URL 将类似于以下内容：

  ```
  ec2-10-221-8-71.us-north-2.compute.amazonaws.com
  ```

  使用此信息通过将地址与端口“7051”并置来构建环境变量 `PEERADDR`，因此 `PEERADDR` 的值与以下值类似：

  ```
  ec2-10-221-8-71.us-north-2.compute.amazonaws.com:7051
  ```
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
    export PEERADDR=ec2-10-221-8-71.us-north-2.compute.amazonaws.com:7051
    ```
    {:codeblock}

3. 访存通道的起源区块以在同级上构建通道分类帐。首先使用 `cd` 切换到根目录，然后运行以下命令来访存起源区块。

  ```
  cd /

  GOPATH=/ CORE_PEER_TLS_ROOTCERT_FILE=/mnt/msp/tls/cacert.pem CORE_PEER_TLS_ENABLED=true CORE_PEER_ADDRESS=${PEERADDR} CORE_PEER_LOCALMSPID=${ORGID} CORE_PEER_MSPCONFIGPATH=/mnt/msp/ GOPATH=/ peer channel fetch 0 -o ${ORDERER_1} -c ${CHANNEL} --cafile /mnt/msp/orderer_tlscacert.pem --tls
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

  在成功添加起源区块后，您应该会看到与以下示例类似的信息：
  ```
  defaultchannel_0.block
  ```
  <!-- removing the code block since this is a result and not an executable command
  {:codeblock}
  -->
4. 通过使用以下命令传递起源区块来将同级加入通道：

  ```
  GOPATH=/ CORE_PEER_TLS_ROOTCERT_FILE=/mnt/msp/tls/cacert.pem CORE_PEER_TLS_ENABLED=true CORE_PEER_ADDRESS=${PEERADDR} CORE_PEER_LOCALMSPID=${ORGID} CORE_PEER_MSPCONFIGPATH=/mnt/msp/ GOPATH=/ peer channel join -b ${CHANNEL}_0.block
  ```
  {:codeblock}

  完成后，应该会看到类似于以下内容的内容：

  ```
  2018-07-06 18:32:09.509 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
  2018-07-06 18:32:09.992 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
  2018-07-06 18:32:09.992 UTC [main] main -> INFO 003 Exiting.....
  ```

### 使用 Fabric Tools 容器在同级上安装链代码
{: #aws-toolcontainer-install-cc}

现在，我们已准备就绪，可以在同级上安装和实例化链代码。对于这些指示信息，将安装 `fabric-samples` 存储库中的 `fabcar` 链代码。使用以下命令从 GitHub 下载 `fabric-samples` 链代码：

  ```
  cd /
  mkdir /src
  cd /src
  git clone https://github.com/hyperledger/fabric-samples
  ```
  {:codeblock}

运行以下同级 CLI 命令将 `fabcar` 链代码安装到同级上。

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
{: #aws-toolcontainer-instantiate-cc}

因为只需要一个同级在通道上实例化链代码，所以不必使用您的同级来实例化链代码。在 {{site.data.keyword.blockchainfull_notm}} Platform 上托管的同级可以执行此操作。但是，如果您希望同级将链代码实例化到通道，那么可以在 Fabric Tools 容器中运行以下命令：

```
CORE_PEER_TLS_ROOTCERT_FILE=/mnt/msp/tls/cacert.pem CORE_PEER_TLS_ENABLED=true CORE_PEER_ADDRESS=${PEERADDR} CORE_PEER_LOCALMSPID=${ORGID} CORE_PEER_MSPCONFIGPATH=/mnt/msp/ GOPATH=/ peer chaincode instantiate -o ${ORDERER_1} -C ${CHANNEL} -n ${CC_NAME} -v v0 -c '{"Args":[""]}' --tls --cafile /mnt/msp/orderer_tlscacert.pem -P ""
```
{:codeblock}

此命令成功完成后，应该会看到类似于下面的内容：

```
2018-07-06 18:43:15.066 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-07-06 18:43:15.066 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-07-06 18:43:50.227 UTC [main] main -> INFO 003 Exiting.....
```


## 重新启动在 AWS 中运行的同级
{: #remote-peer-aws-restart}

您可以在部署同级的环境中启动、停止或重新启动同级。每当向同级添加证书后，都需要重新启动同级，以便识别到新证书。有多种方法可以执行此操作：

- 在 AWS 控制台中，重新引导同级 VPC 实例。
- 从同级容器中，运行以下命令：

 ```
 docker restart peer
 ```
 {:codeblock}  

此外，可以使用 [HEAD 请求](/docs/services/blockchain/monitor_network.html#monitor-nodes)来检查同级的可用性。

## 查看同级日志

通过 SSH 登录 AWS VPC 同级实例，然后运行以下命令来查看同级日志：

```
docker logs peer
```
{:codeblock}

## 更新链代码

一段时间后，您可能需要修改在同级上运行的链代码。由于链代码是安装在同级上并在通道上实例化的，因此需要更新通道上所有同级上的链代码。

完成以下步骤以更新链代码：

1. 要在 AWS 中每个同级上更新链代码，只需使用客户机应用程序或 CLI 命令重新运行用于在同级上安装链代码的过程即可。请确保指定的链代码名称与最初使用的链代码名称相同。但是，这次将递增链代码`版本`号。

2. 在通道上的所有同级上安装新的链代码后，使用“网络监视器”或[同级链代码升级 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/peerchaincode.html#peer-chaincode-upgrade) 命令来更新通道，以使用新的链代码。

有关使用“网络监视器”的“安装代码”面板更新通道上的链代码的更多信息，请参阅这些[指示信息](/docs/services/blockchain/install_instantiate_chaincode.html#updating-a-chaincode)中的步骤 2。

## 故障诊断

### **问题：**远程同级无法连接到区块链网络

堆栈创建成功完成，但 Docker 日志包含以下错误：

```
[main] main -> ERRO 001 Cannot run peer because error when setting up MSP of type bccsp from directory /etc/hyperledger/awspeer1/msp: Setup error: nil conf reference
```

**解决方案：**  
在部署快速启动模板时没有在 CAUrl 中指定端口可能导致此错误。CAUrl 应该类似于 `https://<network>-org1-ca.stage.blockchain.ibm.com:31011`。
请重新部署快速入门模板，同时注意为 CAUrl 指定正确的值。

### **问题：**实例化链代码因错误而失败

链代码实例化失败，错误为：
```
Error: Error endorsing chaincode: rpc error: code = Unknown desc = chaincode error (status: 500, message: instantiation policy violation: signature set did not satisfy policy)
```

**解决方案：**   
确保在将管理员证书上传到“网络监视器”后，在通道上同步证书。请参阅这些[指示信息](#manage-certs)中的步骤 5，以了解更多信息。注意，同级加入通道之前，务必同步通道证书。
