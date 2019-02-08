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

# 部署 {{site.data.keyword.blockchainfull_notm}} Platform for AWS
{: #remote-peer-aws}


***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


以下指示信息描述了如何使用 Amazon Web Service (AWS) 快速入门模板来创建 {{site.data.keyword.blockchainfull}} Platform for AWS 同级，然后将其连接到 {{site.data.keyword.blockchainfull_notm}} Platform 上的网络。
{:shortdesc}

有关 AWS 的更多信息，请参阅 [AWS 概述文档 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://d1.awsstatic.com/whitepapers/aws-overview.pdf "AWS 概述文档")。

部署 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 同级之前，请查看[同级注意事项](/docs/services/blockchain/remote_peer.html#remote-peer-limitations)。

## 先决条件
{: #prerequisites-aws}

要使用 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 同级（同级），您必须有一个组织，并且该组织是在 IBM Blockchain Platform 上托管的区块链网络的成员。您需要使用 IBM Cloud 上的“网络监视器”来访问网络的网络凭证和 API 端点。如果您不是任何区块链网络的成员，那么需要创建或加入一个网络。有关更多信息，请参阅[创建网络](/docs/services/blockchain/get_start.html#creating-a-network)或[加入网络](../get_start.html#joining-a-network)。

同级的缺省 VPC 实例类型为 `m4.xlarge`。您应该根据自己的 CPU、内存和存储需求来优化选择的实例类型。同级至少需要：  
-	2 个 CPU
-	2 GB RAM
-	用于链代码的 4 GB 空间
-	用于分类帐的 10 GB 空间，并能够随着分类帐的扩大而增大。

这些最低资源级别足以用于测试和试验。对于生产环境，请务必分配足够大的存储量，例如 100 GB。使用的存储量取决于事务数和网络中所需的签名数。如果同级上的存储量耗尽，那么必须<!-- either expand the storage or -->在更大的文件系统中部署新的同级，并使其通过相同通道上的其他同级同步。


## 部署选项
{: #remote-peer-aws-deploy-options}

快速入门提供了两个部署选项：

* 将 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 部署到新的 VPC 中（端到端部署）。此选项会构建一个新的 AWS 环境（包含 VPC、子网、NAT 网关、安全组、防御主机和其他基础架构组件），然后将同级部署到这一新的 VPC 中。

* 将 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 部署到现有 VPC 中。此选项在现有 AWS 基础架构中供应 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 同级。对于这些选项，分别有单独的模板，您可在其中配置 CIDR 块、实例类型和同级设置，如本指南后面所述。

## 步骤 1：准备 AWS 帐户
{: #remote-peer-aws-account}

1. 如果您还没有 AWS 帐户，请单击[此处 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://aws.amazon.com "https//aws/amazon.com") 并遵循屏幕上的指示信息进行创建。

2. 使用导航栏中的区域选择器来选择要在 AWS 中部署同级的 AWS 区域。

3. 在首选的区域中创建密钥对。

4. 如果需要，请求增大 Amazon EC2 <type> 实例类型的服务限制。如果您已有使用此实例类型的现有部署，并且您认为可能会超出此部署的缺省限制，那么可能需要执行此操作。

## 步骤 2：检索远程同级配置信息
{: #aws-network-endpoints}

您需要在配置期间向同级提供网络的 API 端点。这些端点允许同级查找并连接到 {{site.data.keyword.blockchainfull_notm}} Platform 上的网络。在“网络监视器”的**概述**屏幕上，单击**远程同级配置**按钮。

![远程同级配置](../images/myresources_starter.png "远程同级配置")
*图 1.“远程同级配置”面板*

这将打开一个弹出窗口，其中显示以下字段的值。保存以下字段的值，在使用 AWS 快速入门模板配置同级时需要这些值。

- **组织 MSP**
- **认证中心 (CA) 名称**
- **认证中心 (CA) URL**
- **认证中心 (CA) TLS 证书**

您可以直接将每个字段复制并粘贴到快速入门模板中，或者通过单击**下载**链接将其保存为 JSON 文件。

**注**：如果以 JSON 格式下载这些信息，那么在将 TLS 证书提供给同级之前，需要先将其转换为 PEM 格式。通过发出以下命令，将下载的 JSON 文件中的**认证中心 (CA) TLS 证书**转换为 PEM 格式：
```
echo -e "<CERT>" > admin.pem
```
{:codeblock}

将 `<CERT>` 替换为**认证中心 (CA) TLS 证书**的值。然后，系统提示您在快速入门模板中提供**认证中心 (CA) TLS 证书**时，请对 admin.pem 文件执行 `cat` 命令，将内容复制并粘贴到该字段中。  

## 步骤 3：注册 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 同级
{: #aws-register-peer}

需要向 {{site.data.keyword.blockchainfull_notm}} Platform 上的网络添加新的同级身份后，{{site.data.keyword.blockchainfull_notm}} Platform for AWS 同级才能加入网络。完成以下步骤以注册同级。

**注：**为了实现高可用性，快速入门模板会在两个可用性专区中创建两个同级节点。因此，需要两个注册标识和私钥。**重复此过程两次，以生成两个注册标识和私钥。**

1. 在 {{site.data.keyword.blockchainfull_notm}} Platform 上登录到网络的“网络监视器”。在“网络监视器”的“认证中心”屏幕上，可以查看已向网络注册的所有身份，例如管理员或客户机应用程序。
  ![CA 屏幕](../images/CA_screen_starter.png "CA 屏幕")
  *图 2. CA 屏幕*

2. 单击面板上的**添加用户**按钮。这将打开一个弹出屏幕，并允许您在填写以下字段后向网络注册同级。**保存“标识”和“私钥”的值，以供后续在快速入门模板中配置同级时使用。**
  - **注册标识：**要用于同级的名称，在配置同级时称为`注册标识`。**保存此值**以供未来使用。
  - **注册私钥：**要用于同级的密码，在配置同级时称为`注册私钥`。**保存此值**以供未来使用。
  - **类型：**对于此字段，选择`同级`。
  - **亲缘关系：**这是同级属于的组织（例如，`org1`）下的亲缘关系。可以指定新的亲缘关系，也可以使用现有亲缘关系。
  - **最大注册数：**您可以使用此字段来限制可使用此身份注册或生成证书的次数。如果未指定，那么值会缺省为无限次数注册。

  填写完字段后，单击**提交**以注册同级。随后，注册的同级会以网络上的身份在表中列出。作为一项安全措施，每个身份以及附带的注册标识和私钥仅用于部署一个同级。请勿复用同级的标识和密码。

## 步骤 4：启动快速入门
{: #remote-peer-aws-launchqs}

在运行此快速入门参考部署时使用的 AWS 服务的成本由您负责。使用此快速入门没有其他成本。有关完整的详细信息，请参阅此快速入门中需要使用的每个 AWS 服务的定价页面。价格可能会随时更改。

1. 选择下列其中一个选项以将 AWS CloudFormation 模板启动到 AWS 帐户中。有关选择选项的帮助，请参阅本指南前面的“部署选项”。每个部署大约需要 10 分钟才能完成。  

  * [将 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 部署到 AWS 上的新 VPC 中 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://fwd.aws/v43nk "将 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 部署到 AWS 上的新 VPC 中")。  

  * [将 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 部署到 AWS 上的现有 VPC 中 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://fwd.aws/zrP4g "将 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 部署到 AWS 上的现有 VPC 中")。

  **重要信息：**     
  如果要将 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 部署到现有 VPC 中，请确保 VPC 有两个公用子网分别位于数据库实例的不同可用性专区中。这两个子网需要其路由表中有 NAT 网关或 NAT 实例，以允许这些实例下载包和软件，而不必将其公开到因特网。您还需要 Amazon VPC 文档中说明的 DHCP 选项中配置的域名选项。  

  另外，请务必创建与现有 VPC 绑定的安全组，并将端口 22 和 7051 上的入站规则添加到此安全组。端口 22 上的 TCP 连接支持对生成的实例进行 SSH 访问，而端口 7051 上的 TCP 连接支持对同级实例进行外部 gRPC 访问（使用 Fabric 工具 CLI 和 Fabric SDK 来操作同级时需要）。启动快速入门时，系统将提示您输入这些 VPC 设置。

2. 检查显示在导航栏右上角的区域，并根据需要进行更改。这是将构建同级的网络基础架构的位置。缺省情况下，模板在美国东部（俄亥俄州）区域中启动。

3. 在“选择模板”页面上，保留模板 URL 的缺省设置，然后选择`下一步`。

4. 在“指定详细信息”页面上，根据需要更改堆栈名称。复查模板的参数。为需要输入的参数提供值。对于其他所有参数，请复查缺省设置并根据需要对其进行定制。完成对参数的复查和定制后，选择`下一步`。

在以下各表中，参数按类别列出，并分别针对两个部署选项进行了描述：

  * [用于将 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 部署到新 VPC 中的参数](#remote-peer-aws-parameters-newvpc)

  * [用于将 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 部署到现有 VPC 中的参数](#remote-peer-aws-parameters-existvpc)

### 用于将 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 部署到新 VPC 中的参数
{: #remote-peer-aws-parameters-newvpc}

下表列出了 AWS 图表的可配置参数及其缺省值。所有值都是必需的。

|参数|描述|缺省值|
| --------------|-------------|---------|
|`堆栈名称`|堆栈名称是一个标识，可帮助您在堆栈列表中找到特定堆栈。堆栈名称只能包含字母数字字符（区分大小写）和连字符。名称必须以字母字符开头，并且长度不能超过 128 个字符。| |
| | | |
|**网络配置**| |
|`可用性专区`|要用于 VPC 中子网的两个可用性专区。注：会保留逻辑顺序。| |
|`允许的 SSH 访问 CIDR`|允许 [CIDR 块 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html#vpc-resize "VPC 和子网") 用于对 IBM Blockchain 同级实例进行外部 SSH 访问。可以将其设置为 0.0.0.0/0，以允许从任何位置进行访问（不建议）。| |
|`PeerEndpointAccessCIDR`|允许 [CIDR ![外部链接图标](../images/external_link.svg "外部链接图标")](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html#vpc-resize "VPC 和子网") 块用于对 IBM Blockchain 同级实例进行外部 gRPC 访问。通常设置为 0.0.0.0/0，以允许从任何位置进行访问（不建议）。| |
| | | |
|**Amazon EC2 配置**| | |
|`InstanceType`|同级实例的 EC2 实例的类型。|m4.xlarge|
|`KeyPairName`|AWS 区域内现有 EC2 密钥对的名称。必须生成此项。| |
| | | |
|**IBM Blockchain 配置**| |
|`IBMBlockchainVersion`|要部署的 IBM Blockchain 版本。|1.2.1|
|`StateDatabase`|要用于存储区块链状态的数据库类型。此选择应该与网络的其余部分使用的状态数据库类型相匹配。|CouchDB|
|`PeerVolumeSize`|用于存储同级的持久数据（分类帐、状态数据库和 MSP）的 EBS 卷大小（以 GB 为单位）。|100 |
|`同级 1 注册标识`|在 IBM Blockchain Platform UI“认证中心”面板中为第一个同级输入的注册标识。|  |
|`同级 1 注册私钥`|在 IBM Blockchain Platform UI“认证中心”面板中为第一个同级输入的注册私钥。| |
|`同级 2 注册标识`|在 IBM Blockchain Platform UI“认证中心”面板中为第二个同级输入的注册标识。| |
|`同级 2 注册私钥`|在 IBM Blockchain Platform UI“认证中心”面板中为第二个同级输入的注册私钥。| |
| | | |
|**IBM Blockchain 服务凭证**| | |
|`组织 MSP`|此值可在 IBM Blockchain Platform UI 中找到。单击“概述”面板上的“远程同级配置”按钮，然后将该信息复制并粘贴到此处。| |
|`认证中心 (CA) 名称`|此值可在 IBM Blockchain Platform UI 中找到。单击“概述”面板上的“远程同级配置”按钮，然后将该信息复制并粘贴到此处。| |
|`认证中心 (CA) URL`|此值可在 IBM Blockchain Platform UI 中找到。单击“概述”面板上的“远程同级配置”按钮，然后将该信息复制并粘贴到此处，包括端口。如果未指定，缺省端口为 443。| |
|`认证中心 (CA) TLS 证书`|此值可在 IBM Blockchain Platform UI 中找到。单击“概述”面板上的“远程同级配置”按钮，然后将该信息复制并粘贴到此处。| |
| | | |
|**其他参数**| | |
|`QSS3BucketName`|快速入门资产的 S3 存储区名称。快速入门存储区名称可以包含数字、小写字母、大写字母和连字符 (-)。不能以连字符 (-) 开头或结尾。|`aws-quickstart`|
|`QSS3KeyPrefix`|快速入门资产的 S3 密钥前缀。快速入门密钥前缀可以包含数字、小写字母、大写字母、连字符 (-) 和正斜杠 (/)。|`quickstart-ibm-fabric/`|

1. 在“选项”页面上，可以为堆栈中的资源指定标记（键值对）并设置高级选项。完成后，选择“下一步”。

2. 在“复查”页面上，复查并确认模板设置。在“功能”下，选中相应复选框以确认模板将创建 IAM 资源。

3. 选择“创建”以部署堆栈。

4. 监视堆栈的状态。当所有堆栈的状态都为 `CREATE_COMPLETE` 时，说明同级集群已准备就绪。成功完成后，如果选择了 CouchDB，那么您应该有包含四个嵌套堆栈的根堆栈；如果选择了 LevelDB，那么您应该有包含两个嵌套堆栈的根堆栈。

5. 使用堆栈的“输出”选项卡中显示的信息来查看已创建的资源。


### 用于将 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 部署到现有 VPC 中的参数
{: #remote-peer-aws-parameters-existvpc}

如果要将 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 同级部署到现有 VPC 中，应考虑以下注意事项：

 - 确保 VPC 有两个专用子网分别位于数据库实例的不同可用性专区中。这两个子网需要其路由表中有 NAT 网关或 NAT 实例，以允许这些实例下载包和软件，而不必将其公开到因特网。

 - 配置 [Amazon VPC 文档 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_DHCP_Options.html "DHCP 选项集") 中说明的 DHCP 选项中的域名选项。  

- 创建与现有 VPC 绑定的安全组，并将端口 22 和 7051 上的入站规则添加到此安全组。端口 22 上的 TCP 连接支持对生成的实例进行 SSH 访问，而端口 7051 上的 TCP 连接支持对同级实例进行外部 gRPC 访问（使用 Fabric 工具 CLI 和 Fabric SDK 来操作同级时需要）。启动快速入门时，系统将提示您输入这些 VPC 设置。

 将 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 同级部署到现有 VPC 中时，以下参数将替换[上面](#remote-peer-aws-parameters-newvpc)相应部分中的参数：

|参数|描述|缺省值|
| --------------|-------------|---------|
|**网络配置**| | |
|`VPCID`|	用于部署的现有 VPC 的标识。| |
|`AvailabilityZone1`|要在其中部署第一个同级节点的可用性专区。| |
|`SubnetID1`|	要用于第一个同级节点的子网的标识。子网应该属于所选的 VPC。要查找属于 VPC 的子网，请查看 AWS VPC 仪表板，然后选择“子网”菜单。| |
|`AvailabilityZone2`|要在其中部署第二个同级节点的可用性专区。| |
|`SubnetID2`|	要用于第二个同级节点的子网的标识。子网应该属于所选的 VPC。要查找属于 VPC 的子网，请查看 AWS VPC 仪表板，然后选择“子网”菜单。| |
| | | |
|**Amazon EC2 配置**| | |
|`InstanceType`|同级实例的 EC2 实例的类型。|m4.xlarge|
|`KeyPairName`|	AWS 区域内现有 EC2 密钥对的名称。必须生成此项。| |
|`SecurityGroup`|AWS 区域内现有 EC2 安全组的标识。您应该允许端口 22 和 7051 上的入站 TCP 连接。|	| |

## 步骤 5：测试部署
{: #remote-peer-aws-test}

AWS CloudFormation 模板成功创建堆栈后，AWS 帐户中将有两个 {{site.data.keyword.blockchainfull_notm}} Platform for AWS 同级实例在运行。实例的命名将基于快速入门模板中指定的`组织 MSP` 和`同级注册标识`的组合。例如，`org1-remotepeer1`。  

![同级 AWS EC2 实例](../images/remote_peer_AWS_EC2_instances.png "同级 AWS EC2 实例")  
*图 3. AWS EC2 实例上的远程同级*

要验证同级是否正在运行，请执行以下操作：

  * 在 AWS 控制台中选择实例（单击**服务 > EC2 > 实例**），然后单击**连接**按钮，以通过 SSH 登录到新创建的 VPC。遵循 AWS 中的指示信息发出 `ssh` 命令。  
  * 在命令行中，运行 `docker ps` 以查看正在运行的容器。每个虚拟机都包含一个同级容器和一个 ecs-agent。如果选择了 CouchDB 作为分类帐数据库，那么还将有一个 CouchDB 容器。

  ```
  CONTAINER ID        IMAGE                                STATUS              PORTS                          NAMES
  fb3c49fe52fe        amazon/amazon-ecs-agent:latest       Created                                            ecs-agent
  667780cf3cd3        ibmblockchain/fabric-peer:1.2.1      Up                  0.0.0.0:7051->7051/tcp         peer
  2aa143c81027        ibmblockchain/fabric-couchdb:0.4.6   Up                  4369/tcp, 5984/tcp, 9100/tcp   couchdb
  ```

  * 您可以通过运行 `docker exec -it peer sh` 在同级容器内创建 shell 会话。

此外，要验证同级连接是否对 {{site.data.keyword.blockchainfull_notm}} Platform 网络有效，可以从同级容器内运行同级 CLI 命令。运行 `peer channel fetch` CLI 命令以访问通道中的起源区块：

1. 在“网络监视器”的“概述”面板中提供的`连接概要文件`中检索配置信息。单击**连接概要文件**，然后单击**下载**。

   -  通过搜索`排序者 > URL` 下的**排序者**来查找排序者 URL。记下以网络名称开头的 URL。该 URL 类似于以下示例：
    

   ```
    ash-zbc07b.4.secure.blockchain.ibm.com:21239
    ```

   - 通过搜索**组织**来查找组织的名称。这应该是您用于注册同级的组织。您可以找到组织的名称及其关联的 `mspid`。此值还在“网络监视器”的“概述”面板中提供。单击**远程同级配置**按钮。此值会显示在`组织 MSP` 下。请记下 `mspid` 的值。

   - 如果尚未通过运行 `docker exec -it peer sh` 在同级容器内创建 shell 会话，请进行创建。

   ```
   docker exec -it peer sh
   ```
   {:codeblock}

   - 将排序节点 TLS 证书从连接概要文件复制到同级。浏览至**排序节点**部分。复制证书中位于 "pem:" 后面的内容，从 -----BEGIN CERTIFICATE----- 开始，到 -----END CERTIFICATE----- 结束。不要包含引号。在命令行中运行以下命令，并将 `<orderer cert>` 替换为从 creds.json 文件中复制的内容。

   ```
   echo -e "<orderer cert>" > /etc/hyperledger/<PEER_ENROLL_ID>/orderer_tlscacert.pem
   ```
   {:codeblock}

   将 `<PEER_ENROLL_ID>` 替换为在快速入门模板中指定且与此同级实例关联的注册标识。

2. 您需要先将组织添加到网络中的通道，然后才能访存起源区块。

  - 可以为同级启动新的通道。作为通道启动者，您可以在[通道创建](/docs/services/blockchain/create_channel.html#creating-a-channel)期间自动包含您的组织。

  - 区块链网络的其他成员也可以使用[通道更新](/docs/services/blockchain/create_channel.html#updating-a-channel)将您的组织添加到现有通道。

  - 将组织添加到通道后，需要将同级的签名证书添加到该通道。在安装期间，同级将上传其签名证书，因此您只需要将该证书同步到通道即可。在“网络监视器”的“通道”屏幕中，找到组织加入的通道，然后从**操作**标头下的下拉列表中选择**同步证书**。此操作会同步通道上所有同级的证书。

3. 运行以下命令在同级容器中设置环境变量。

   ```
   export ORDERER_1=<ORDERER_URL>
   export CHANNEL=<CHANNEL_NAME>
   export ORGID=<ORGANIZATION_MSP_ID>
   export PEERADDR=<PEER_ADDR>
   ```
   {:codeblock}

   将相关字段替换为您自己的信息。
     - 将 `<ORDERER_URL>` 替换为 `creds.json` 文件中排序者的主机名和端口。
     - 将 `<CHANNEL_NAME>` 替换为同级将加入的通道的名称。
     - 将 `<ORGANIZATION_MSP_ID>` 替换为 `creds.json` 文件中组织的名称。
     - 将 `<PEER_ADDR>` 替换为 `localhost:7051`。

   例如：

   ```
   export ORDERER_1=ash-zbc07b.4.secure.blockchain.ibm.com:21239
   export CHANNEL=defaultchannel
   export ORGID=PeerOrg1
   export PEERADDR=localhost:7051
   ```

4. 运行以下同级 CLI 命令以访存通道的起源区块。

   **重要信息：**在以下命令中，将出现的每个 `<PEER_ENROLL_ID>` 替换为在快速入门模板中指定且与此同级实例关联的注册标识。可以通过运行 `ls /etc/hyperledger/` 命令来找到此值。这将列出两个文件夹：一个是 `fabric`，另一个是您的 `<PEER_ENROLL_ID>`。

   ```
   CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/<PEER_ENROLL_ID>/tls/ca.crt CORE_PEER_TLS_ENABLED=true CORE_PEER_ADDRESS=${PEERADDR} CORE_PEER_LOCALMSPID=${ORGID} CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/<PEER_ENROLL_ID>/msp/ GOPATH=/ peer channel fetch 0 -o ${ORDERER_1} -c ${CHANNEL} --cafile /etc/hyperledger/<PEER_ENROLL_ID>/orderer_tlscacert.pem --tls
   ```
   {:codeblock}


   **注：**运行其中任一 CLI 命令时，可能会遇到以下警告，您可以安全地忽略此警告。

   ```
   [msp] getPemMaterialFromDir -> WARN 001 Failed reading file
   /etc/hyperledger/<PEER_ENROLL_ID>/msp/intermediatecerts/<intermediate cert name>.pem: no pem content for file  /etc/hyperledger/<PEER_ENROLL_ID>/msp/intermediatecerts/<intermediate cert name>.pem
   ```

   通过运行以下命令来验证该命令是否已成功运行，以及是否将起源区块添加到了同级容器：

   ```
  ls *.block
  ```
   {:codeblock}

   您看到类似于以下示例的内容时，即知道成功添加了起源区块：

   ```
  defaultchannel_0.block
  ```

   祝贺您。{{site.data.keyword.blockchainfull_notm}} Platform for AWS 同级已成功连接到 {{site.data.keyword.blockchainfull_notm}} Platform 网络。

## 常见问题
{: #remote-peer-aws-faq}

* **问**：我在启动快速入门时，遇到了 CREATE_FAILED 错误。
* **答**：如果 AWS CloudFormation 创建堆栈失败，建议您将“发生故障时回滚”设置为`否`并重新启动该模板。（此设置位于 AWS CloudFormation 控制台的“选项”页面中的“高级”下。）使用此设置时，将保留堆栈的状态，并且实例将保持运行状态，以便您可以对该问题进行故障诊断。（请查看 `%ProgramFiles%\Amazon\EC2ConfigService` 和 `C:\cfn\log` 中的日志文件。）

  - 将“发生故障时回滚”设置为`否`时，此堆栈会继续产生 AWS 费用。完成故障诊断后，请确保删除此堆栈。有关其他信息，请参阅 AWS Web 站点上的 [Troubleshooting AWS CloudFormation ![外部链接图标](../images/external_link.svg "外部链接图标")](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/troubleshooting.html "Troubleshooting AWS CloudFormation")。

* **问**：我在部署 AWS Cloudformation 模板时，遇到了大小限制错误。
* **答**：建议您从我们提供的位置或从其他 S3 存储区启动快速入门模板。如果从计算机上的本地副本或从非 S3 位置部署模板，那么在创建堆栈时可能会遇到模板大小限制。有关 AWS CloudFormation 限制的更多信息，请参阅 [AWS 文档 ![外部链接图标](../images/external_link.svg "外部链接图标")](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-limits.html "AWS CloudFormation Limits")。

## 下一步做什么
{: #whats-next-aws}

在 AWS 中设置同级后，可以完成若干操作步骤，然后才能将事务提交到区块链网络的分布式分类帐，并读取该分布式分类帐。有关更多信息，请参阅[在 AWS 中操作同级](/docs/services/blockchain/remote_peer_operate_aws.html#remote-peer-operate-aws)。

## 高可用性 (HA)
{: #aws-high-availability}

缺省情况下，为了支持 HA，快速入门模板会将同级的两个实例部署在两个不同的可用性专区中。要利用此 HA 支持，您还需要配置[客户机应用程序以实现高可用性](/docs/services/blockchain/v10_application.html#ha-app)。

## 安全注意事项
{: #remote-peer-aws-security}

AWS Cloud 提供了一个可扩展且高度可靠的平台，可帮助客户快速、安全地部署应用程序和数据。在 AWS 基础架构上构建系统时，安全责任由您和 AWS 共担。这种共担模式可以减轻您的操作负担，因为 AWS 会负责操作、管理和控制主机操作系统中的组件、虚拟化层以及向下至运行服务的设施的物理安全性。而您则负责管理访客操作系统（包括更新和安全补丁）、管理其他关联的应用程序以及配置 AWS 提供的安全组防火墙。有关 AWS 安全性的更多信息，请访问 [AWS 云安全性 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://aws.amazon.com/security/ "AWS 云安全性")。

### AWS Identity and Access Management (IAM)
{: #remote-peer-aws-iam}

此解决方案利用具有最少特权访问权的 IAM 角色。不需要或不建议您在供应的实例上存储 SSH 密钥、密钥或访问密钥。

### 操作系统安全性
{: #remote-peer-aws-ossecurity}

只能使用在部署过程中指定的 SSH 密钥来访问集群节点上的 root 用户。AWS 不会存储这些 SSH 密钥，因此，如果丢失了 SSH 密钥，那么您可能会失去对这些实例的访问权。执行操作系统补丁是您的责任，应定期执行。

### 安全组
{: #remote-peer-aws-securitygroups}

安全组充当控制一个或多个实例的流量的防火墙。启动实例时，可将一个或多个安全组与该实例相关联。您可向每个安全组添加规则，以允许进出其关联实例的流量。您可以随时修改安全组的规则。新规则会自动应用于与安全组关联的所有实例。将尽可能限制作为此解决方案的一部分创建并分配给单个实例的安全组，而允许访问同级所需的各种功能。我们建议在集群启动并开始运行后，根据需要复查安全组以进一步限制访问权。

### 同级安全性
{: #aws-security}

同级部署在 {{site.data.keyword.blockchainfull_notm}} Platform 外部；因此，您负责管理同级的安全性。这包括企业套餐网络提供的重要安全性方面，例如密钥管理和数据加密。考虑同级的安全性时，请查看以下主题。

#### 数据安全性
{: #aws-security-data}

{{site.data.keyword.blockchainfull_notm}} Platform 企业套餐使用的是基于[对称密钥加密 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSB23S_1.1.0.14/gtps7/s7symm.html "对称加密") 的全盘加密来保护网络使用的所有数据。您必须在自己的环境中执行类似的步骤来保护同级数据。

状态数据库（无论使用的是 LevelDB 还是 CouchDB）中的数据未加密。可以使用应用程序级别加密来保护状态数据库中的静态数据。

#### 数据存储位置
{: #aws-security-data-residency}

数据存储位置要求所有区块链分类帐数据的处理和存储始终限制在一个国家或地区境内。有关如何实现此操作的更多详细信息，请参阅此[主题](/docs/services/blockchain/remote_peer.html#data-residency)。

#### 密钥管理
{: #aws-security-key-management}

密钥管理是同级安全性中至关重要的一个方面。如果专用密钥泄漏或丢失，那么恶意参与者可能有能力访问您同级的数据和功能。{{site.data.keyword.blockchainfull_notm}} Platform 企业套餐使用[硬件安全模块](/docs/services/blockchain/glossary.html#hsm) (HSM) 来存储网络的专用密钥。HSM 是一种物理设备，用于阻止其他参与方访问您的专用密钥。
在 AWS 上部署同级时，您负责管理专用密钥。虽然 {{site.data.keyword.blockchainfull_notm}} Platform 会生成专用密钥，但这些密钥不会存储在 Platform 上。因此，必须确保将密钥存储在安全的位置，避免密钥泄露。您可以在同级 MSP 的 keystore 文件夹（位于同级容器内的 `/etc/hyperledger/<PEER_ENROLL_ID>/msp/keystore/` 目录）中找到同级的专用密钥。有关同级内的证书的更多信息，请访问[在 {{site.data.keyword.blockchainfull_notm}} Platform 上管理证书](/docs/services/blockchain/certificates.html)主题中的[成员资格服务提供者](/docs/services/blockchain/certificates.html#msp)部分。

您可以使用“密钥代管”来恢复丢失的专用密钥。这需要在丢失任何密钥之前执行。如果无法恢复专用密钥，那么需要通过从认证中心获取新的 signCert 来获取新的专用密钥。您还应该从加入的任何通道中除去并替换管理员证书。

<!---
In IBP when a private key is created, two sets of independent key material is generated, in custody of two different entities. Those two sets of key materials are then combined to create the private key.
--->

#### TLS
{: #aws-security-tls}

[传输层安全性 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_7.1.0/com.ibm.mq.doc/sy10660_.htm "SSL 或 TLS 握手概述") (TLS) 嵌入在 Hyperledger Fabric 的信任模型中。{{site.data.keyword.blockchainfull_notm}} Platform 上的所有组件使用 TLS 进行认证并相互通信。因此，{{site.data.keyword.blockchainfull_notm}} Platform 上的网络组件需要能够完成与同级的 TLS 握手。这样做的其中一个影响是，您需要在从客户机应用程序到同级之间的防火墙中启用传递（例如，使用白名单来实现）。


#### 成员资格服务提供者配置
{: #aws-security-MSP}

IBM Blockchain Platform 的组件通过成员资格服务提供者 (MSP) 使用身份。MSP 将 CA 颁发的证书与网络和通道角色相关联。请参阅此[主题](/docs/services/blockchain/certificates.html#msp)，以获取有关 MSP 如何用于同级的更多信息。

#### 应用程序安全性
{: #aws-security-appl}

由于所有链代码调用都已签名，因此 Fabric 会管理应用程序安全性。此外，Fabric 还包含基于 ACL 的应用程序级别检查。

## 许可和定价
{: #license-pricing-aws}

您必须接受 {{site.data.keyword.blockchainfull_notm}} Platform for AWS Community Edition 许可证版本，才能使用快速入门支持的部署解决方案。{{site.data.keyword.blockchainfull_notm}} Platform for AWS（包括通过快速入门产品提供的所有包以及从这些包中派生的包）不适合用于生产用途。IBM 可能会随时决定撤销授权对代码的访问权以及停止使用此代码。{{site.data.keyword.blockchainfull_notm}} Platform for AWS 软件许可协议包含有关许可条款的更多详细信息。启动快速入门时，将要求您阅读并同意协议的条款。
