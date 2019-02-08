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

# 在 {{site.data.keyword.cloud_notm}} Private 中部署同级并连接到入门套餐或企业套餐
{: #peer-ibp}


***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

以下指示信息描述了如何在 {{site.data.keyword.cloud_notm}} Private (ICP) 上部署 {{site.data.keyword.blockchainfull}} Platform 同级，该同级将连接到 {{site.data.keyword.cloud_notm}} 或您本地 ICP 上的入门套餐或企业套餐网络。
{:shortdesc}

部署同级之前，请查看[注意事项和限制](/docs/services/blockchain/ibp-for-icp-about.html#ibp-icp-considerations)。

入门套餐或企业套餐网络必须运行的是 Hyperledger Fabric V1.1 或 V1.2.1。可以通过打开“网络监视器”中的[“网络首选项”窗口](/docs/services/blockchain/v10_dashboard.html#network-preferences)来找到 Hyperledger Fabric 版本。

## 需要的资源
{: #peer-resources-required}

确保 ICP 系统满足最低硬件资源需求：

|组件|vCPU|RAM|用于数据存储的磁盘|
|-----------|------|-----|-----------------------|
|同级|2|2 GB|50 GB，能够扩展|
|用于同级的 CouchDB|2|2 GB|50 GB，能够扩展|

 **注：**
 - vCPU 是分配给虚拟机的虚拟核心，或者如果服务器没有针对虚拟机进行分区，那么 vCPU 是物理处理器核心。决定用于 ICP 中部署的虚拟处理器核心 (VPC) 时，需要考虑 vCPU 需求。VPC 是确定 IBM 产品许可成本的计量单位。有关决定 VPC 的场景的更多信息，请参阅 [Virtual processor core (VPC) ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SS8JFY_9.2.0/com.ibm.lmt.doc/Inventory/overview/c_virtual_processor_core_licenses.html)。
 - 这些最低资源级别足以用于测试和试验。对于具有大量交易的环境，请务必分配足够大的存储量；例如，为同级分配 250 GB。要使用的存储量取决于交易数和网络中所需的签名数。如果同级上的存储量即将耗尽，那么必须在更大的文件系统中部署新的同级，并使其通过相同通道上的其他组件同步。

## 存储
{: #storage}

您需要确定同级将使用的存储器。如果使用缺省设置，Helm 图表将创建两个新的 8 Gi 持久卷声明 (PVC)，一个 PVC 的名称为 `my-data-pvc`，用于存储同级数据；另一个 PVC 的名称为 `statedb-pvc`，用于存储状态数据库。

如果不想使用缺省存储设置，请确保在 ICP 安装期间设置*新*的 `storageClass`，否则 Kubernetes 系统管理员需要在您部署之前创建 storageClass。

可以选择在 amd64 或 s390x 平台上部署同级。但是，请注意，[动态供应](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/)仅可用于 ICP 中的 amd64 节点。如果集群包含 s390x 和 amd64 工作程序节点的混合，那么无法使用动态供应。

如果未使用动态供应，那么必须创建[持久卷 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)，并对其设置标签，标签可用于优化 Kubernetes PVC 绑定过程。

## 部署同级的先决条件
{: #prerequisites-peer-ibp}

1. 必须先[安装 ICP](/docs/services/blockchain/ICP_setup.html) 并[安装 {{site.data.keyword.blockchainfull_notm}} Platform Helm 图表](helm_install_icp.html)，然后才能在 ICP 上安装同级。

2. 如果使用 Community Edition 并且希望在没有因特网连接的 ICP 集群上运行此 Helm 图表，那么需要在连接因特网的机器上创建归档，然后才能在 ICP 集群上安装归档。有关更多信息，请参阅[向没有因特网连接的集群中添加精选应用程序 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/app_center/add_package_offline.html "向没有因特网连接的集群中添加精选应用程序"){:new_window}。请注意，您可以在 Helm 图表中的 ibm-blockchain-platform-dev/ibm_cloud_pak 下找到规范文件 manifest.yaml。

3. 您必须在 {{site.data.keyword.cloud_notm}} 上具有属于入门套餐或企业套餐网络的组织。同级利用 {{site.data.keyword.blockchainfull_notm}} Platform 网络的 API 端点、Hyperledger Fabric CA 和排序服务来运行。如果您不是任何区块链网络的成员，那么需要创建或加入一个网络。有关更多信息，请参阅[创建网络](../get_start.html#creating-a-network)或[加入网络](../get_start.html#joining-a-network)。

4. 必须首先在 ICP 上[部署 CA](CA_deploy_icp.html)。您将使用此 CA 作为 TLS CA。部署同级之前，请执行[在 ICP 上运行 CA](CA_operate.html#prerequisites) 的必备步骤。除了这些步骤之外，不需要执行其他操作。

5. 在 ICP 控制台中检索 TLS CA 的集群代理 IP 地址的值。**注：**您需要是[集群管理员 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/user_management/assign_role.html "集群管理员角色和操作") 才能访问代理 IP。登录到 ICP 集群。在左侧导航面板中，单击**平台**，然后单击**节点**以查看在集群中定义的节点。单击具有角色`代理`的节点，然后从表中复制`主机 IP` 的值。**重要信息：**请保存此值，配置 Helm 图表的`代理 IP` 字段时会使用此值。

6. 创建同级配置文件，并将其作为 Kubernetes 私钥存储在 ICP 中。您可以在[下一部分](#peer-config-file)中找到创建此文件的步骤。

## 构建配置文件
{: #peer-config-file}

部署同级之前，您需要创建配置 JSON 文件，其中包含有关 {{site.data.keyword.cloud_notm}} 上同级身份和您的认证中心的重要信息。然后，您需要在配置期间使用 [Kubernetes Secret ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/configuration/secret/) 对象将此文件传递到 Helm 图表。通过此文件，同级可从 {{site.data.keyword.cloud_notm}} 上的认证中心获取加入入门套餐或企业套餐网络所需的证书。此文件还包含管理员证书，以允许您以管理员用户身份操作同级。

在以下指示信息中，我们将提供模板 JSON，您可以编辑该模板并将其保存到本地文件系统。然后，我们将指导您如何使用 CA 来完成配置文件。

### 配置文件

下面是配置文件的模板：
```
{
	"enrollment": {
		"component": {
			"cahost": "",
			"caport": "",
			"caname": "",
			"catls": {
				"cacert": ""
			},
			"enrollid": "",
			"enrollsecret": "",
			"admincerts": [""]
		},
		"tls": {
			"cahost": "",
			"caport": "",
			"caname": "",
			"catls": {
				"cacert": ""
			},
			"enrollid": "",
			"enrollsecret": "",
			"csr": {
				"hosts": [""]
			}
		}
	}
}
```
{:codeblock}

将这整个文件复制到文本编辑器中，您可以在其中编辑该文件并将其作为 JSON 文件保存到本地文件系统中。您只需要填写此文件的前两个部分：`"enrollment"` 和 `"tls"`。

要完成配置文件，您需要通过入门套餐或企业套餐网络的“网络监视器”完成若干步骤。

1. [检索入门套餐或企业套餐 CA 的端点信息](#ibp-ca-endpoint)。
2. [向 CA 注册同级](#register-peer)。
3. [注册同级管理员](#register-admin)，以用于操作同级。如果您已经注册了管理员来部署其他同级，那么无需完成此步骤。

您还需要使用在 ICP 上部署的 Fabric CA 客户机和 TLS CA 来完成若干步骤。

1. 使用 Fabric CA 客户机[生成同级管理员的 MSP 文件夹](#enroll-admin)。
2. [检索 TLS CA 的端点信息](#tls-ca-endpoint)。
3. 使用 Fabric CA 客户机[向 TLS CA 注册同级](#tls-register-peer)。


### 入门套餐或企业套餐 CA 信息
{: #ibp-ca-endpoint}

首先，我们需要向配置文件提供 {{site.data.keyword.cloud_notm}} 上 CA 的连接信息。登录到入门套餐或企业套餐的“网络监视器 UI”。在“网络监视器”的**概述**屏幕上，单击**远程同级配置**按钮。这将打开一个弹出窗口，其中包含有关 CA 的必需信息。

![远程同级配置](../images/myresources_starter.png "远程同级配置")
*图 1.“远程同级配置”面板*

弹出窗口包含以下字段：

  - **组织 MSP**
  - **认证中心 (CA) 名称**
  - **认证中心 (CA) URL**
  - **认证中心 (CA) TLS 证书**

在文件中的 `"components"` 部分下，输入来自以上字段的以下值：
- `"caname"` 是**认证中心 (CA) 名称**的值
- `"cahost"` 是 CA URL 的主机名。例如，如果 CA URL 为 `https://ncaca9b06047b4bee966b3dec0cbb6671-org1-ca.stage.blockchain.ibm.com:31011`，那么 `"cahost"` 的值将为 `ncaca9b06047b4bee966b3dec0cbb6671-org1-ca.stage.blockchain.ibm.com`
- `"caport"` 是 `"cahost"` 中的端口。例如，如果 CA URL 为 `https://ncaca9b06047b4bee966b3dec0cbb6671-org1-ca.stage.blockchain.ibm.com:31011`，那么 `"caport"` 将为 `31011`。
- `"cacert"` 是**认证中心 (CA) TLS 证书**字段中的值。在文件中插入证书之前，需要通过运行以下命令并将字符串 `<paste in Certificate Authority (CA) TLS Certificate>` 替换为从“网络监视器”中复制的值，以将其编码为 Base64 格式。

  ```
  export FLAG=$(if [ "$(uname -s)" == "Linux" ]; then echo "-w 0"; else echo "-b 0"; fi)
  echo -e '<paste in Certificate Authority (CA) TLS Certificate>' | base64 $FLAG
  ```
  {:codeblock}

  **注：**以上命令生成的字符串的格式务必为一行。应该类似于以下内容：

  ```
  LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tDQpNSUlFbERDQ0EzeWdBd0lCQWdJUUFmMmo2MjdLZGNpSVE0dHlTOCs4a1RBTkJna3Foa2lHOXcwQkFRc0ZBREJoDQpNUXN3Q1FZRFZRUUdFd0pWVXpFVk1CTUdBMVVFQ2hNTVJHbG5hVU5sY25RZ1NXNWpNUmt3RndZRFZRUUxFeEIzDQpkM2N1WkdsbmFXTmxjblF1WTI5dE1TQXdIZ1lEVlFRREV4ZEVhV2RwUTJWeWRDQkhiRzlpWVd3Z1VtOXZkQ0JEDQpRVEFlRncweE16QXpNRGd4TWpBd01EQmFGdzB5TXpBek1EZ3hNakF3TURCYU1FMHhDekFKQmdOVkJBWVRBbFZUDQpNUlV3RXdZRFZRUUtFd3hFYVdkcFEyVnlkQ0JKYm1NeEp6QWxCZ05WQkFNVEhrUnBaMmxEWlhKMElGTklRVElnDQpVMlZqZFhKbElGTmxjblpsY2lC
  ```

  而不能如下所示：
  ```
  LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tDQpNSUlFbERDQ0EzeWdBd0lCQWdJUUFmMmo2MjdL
  ZGNpSVE0dHlTOCs4a1RBTkJna3Foa2lHOXcwQkFRc0ZBREJoDQpNUXN3Q1FZRFZRUUdFd0pWVXpF
  Vk1CTUdBMVVFQ2hNTVJHbG5hVU5sY25RZ1NXNWpNUmt3RndZRFZRUUxFeEIzDQpkM2N1WkdsbmFX
  VEFlRncweE16QXpNRGd4TWpBd01EQmFGdzB5TXpBek1EZ3hNakF3TURCYU1FMHhDekFKQmdOVkJB
  WVRBbFZUDQpNUlV3RXdZRFZRUUtFd3hFYVdkcFEyVnlkQ0JKYm1NeEp6QWxCZ05WQkFNVEhrUnBa
  ```

将生成的字符串粘贴到文件中 `"catls"` 下的 `"cacert"` 字段中。更新后，`"cacert"` 字段类似于以下示例：

  ```
  "catls": {
    "cacert": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUVsRENDQTN5Z0F3SUJBZ0lRQWYyajYyN0tkY2lJUTR0eVM4KzhrVEFOQmdrcWhraUc5dzBCQVFzRkFBkOHRiUWsKQ0FVdzdDMjlDNzlGdjFDNXFmUHJtQUVTcmNpSXhwZzBYNDBLUE1icDFaV1ZiZDQ9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0KCg=="
  ```

### 注册同级
{: #register-peer}

要将同级加入通道并安装和实例化链代码，您需要首先在 {{site.data.keyword.cloud_notm}} 上向 CA 注册该同级。然后，同级部署可以生成同级加入入门套餐或企业套餐网络所必需的证书。完成以下步骤，以使用`注册标识`和`注册私钥`来注册同级。您应将这两个值粘贴到配置文件中。

1. 登录到 {{site.data.keyword.blockchainfull_notm}} Platform 并访问“网络监视器”。在“认证中心”屏幕上，可以查看已向组织注册的所有身份，例如管理员或客户机应用程序。
  ![CA 屏幕](../images/CA_screen_starter.png "CA 屏幕")
  *图 2. CA 屏幕*

2. 单击屏幕上的**添加用户**按钮。这将打开一个弹出屏幕，并允许您在填写以下字段后注册同级。保存“标识”和“私钥”的值，在配置同级时需要使用这些值。
  - **注册标识：**同级的用户名，在配置同级时称为`注册标识`。**保存此值**以用于配置文件。
  - **注册私钥：**同级的密码，在配置同级时称为`注册私钥`。**保存此值**以用于配置文件。
  - **类型：**对于此字段，选择`同级`。
  - **亲缘关系：**这是同级将属于的组织（例如，`org1`）下的亲缘关系。从下拉列表中选择现有亲缘关系，或输入新的亲缘关系。
  - **最大注册数：**可以使用此字段来限制可以通过此身份注册或生成证书的次数。如果未指定，那么值会缺省为无限次数注册。

  填写完字段后，单击**提交**以注册同级。随后，注册的同级会以组织中的身份在表中列出。作为一项安全措施，每个身份以及附带的注册标识和私钥仅用于部署一个同级。请勿复用同级的标识和密码。

3. 在配置文件中的 `"components"` 部分下，输入以下值：
  - `"enrollid"` 是上一步中`注册标识`的值。
  - `"enrollsecret"` 是上一步中`注册私钥`的值。


### 创建管理员
{: #register-admin}

注册同级身份之后，还需要创建将用于操作同级的管理员身份。您首先需要向 CA 注册此新身份，并将其用于生成 MSP 文件夹。然后，将管理员用户的 signCert 添加到配置文件，在部署期间此证书将成为同级的管理员证书。这允许您使用管理员身份的证书来操作区块链网络，例如在同级上启动新的通道或安装链代码。

您只需要针对属于您组织的组件创建一个管理员身份。如果要部署多个同级，那么只需要完成这些步骤一次。您可以使用为一个同级生成的 signCert 来部署属于您组织的其他任何同级。

1. 登录到 {{site.data.keyword.blockchainfull_notm}} Platform 并访问“网络监视器”。在“认证中心”屏幕上，可以查看已向组织注册的所有身份，例如管理员或客户机应用程序。
  ![CA 屏幕](../images/CA_screen_starter.png "CA 屏幕")
  *图 2. CA 屏幕*

2. 单击屏幕上的**添加用户**按钮。这将打开一个弹出屏幕，并允许您在填写以下字段后注册同级。保存“标识”和“私钥”的值，在配置同级时需要使用这些值。
  - **注册标识：**同级管理员的用户名，在配置同级时称为`注册标识`。**保存此值**以供在生成管理员的 MSP 文件夹时使用。
  - **注册私钥：**同级管理员的密码，在配置同级时称为`注册私钥`。**保存此值**以供在生成管理员的 MSP 文件夹时使用。
  - **类型：**对于此字段，选择`同级`。
  - **亲缘关系：**这是同级将属于的组织（例如，`org1`）下的亲缘关系。从下拉列表中选择现有亲缘关系，或输入新的亲缘关系。
  - **最大注册数：**可以使用此字段来限制可以通过此身份注册或生成证书的次数。如果未指定，那么值会缺省为无限次数注册。

  输入这些字段后，单击**提交**以创建管理员。随后，创建的管理员会以组织中的身份在表中列出。

### 生成同级管理员的 MSP 文件夹
{: #enroll-admin}

注册管理员身份后，需要生成同级管理员的 MSP 文件夹和 signCert。因此，需要对入门套餐或企业套餐 CA 运行 enroll 命令。

1. 如果尚未有 [Fabric CA 客户机](CA_operate.html#fabric-ca-client)，请进行下载。
2. 浏览至用于存储加密资料的目录，并创建将用于存储同级管理员的 MSP 文件夹的文件夹。

  ```
  cd fabric-ca-client
  mkdir peer-admin
  ```
  {:codeblock}

3. 将客户机可以在其中创建证书的路径设置为 `$FABRIC_CA_CLIENT_HOME`。请确保除去早先可能尝试创建的配置资料。如果之前未运行过 `enroll` 命令，那么 `msp` 文件夹和 `.yaml` 文件不存在。

  ```
  export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca-client/peer-admin
  rm -rf $FABRIC_CA_CLIENT_HOME/fabric-ca-client-config.yaml
  rm -rf $FABRIC_CA_CLIENT_HOME/msp
  ```
  {:codeblock}

4. 从“网络监视器”中的“概述”面板打开**连接概要文件** JSON 文件，并找到以下变量：
  - CA 的 URL：`certificateAuthorities` 下的 `url`
  - CA 名称：`caName`

5. 根据使用的服务套餐、位置和集群，从 {{site.data.keyword.cloud_notm}} 下载 TLS 证书。您可以根据认证中心 URL 的域名来查找集群：例如，`us01.blockchain.ibm.com:31011` 或 `us02.blockchain.ibm.com:31011`。

  - 入门套餐的根 TLS 证书
    - 美国：[us01.blockchain.ibm.com.cert ![外部链接图标](../images/external_link.svg "外部链接图标")](https://blockchain-certs.mybluemix.net/us01.blockchain.ibm.com.cert "us01.blockchain.ibm.com.cert")；[us02.blockchain.ibm.com.cert ![外部链接图标](../images/external_link.svg "外部链接图标")](https://blockchain-certs.mybluemix.net/us02.blockchain.ibm.com.cert "us02.blockchain.ibm.com.cert")
    - 英国：[uk01.blockchain.ibm.com.cert ![外部链接图标](../images/external_link.svg "外部链接图标")](https://blockchain-certs.mybluemix.net/uk01.blockchain.ibm.com.cert "uk01.blockchain.ibm.com.cert")；[uk02.blockchain.ibm.com.cert ![外部链接图标](../images/external_link.svg "外部链接图标")](https://blockchain-certs.mybluemix.net/uk02.blockchain.ibm.com.cert "uk02.blockchain.ibm.com.cert")
    - 悉尼：[aus01.blockchain.ibm.com.cert ![外部链接图标](../images/external_link.svg "外部链接图标")](https://blockchain-certs.mybluemix.net/aus01.blockchain.ibm.com.cert "aus01.blockchain.ibm.com.cert")<!--；[aus02.blockchain.ibm.com.cert ![外部链接图标](../images/external_link.svg "外部链接图标")](https://blockchain-certs.mybluemix.net/aus02.blockchain.ibm.com.cert "aus02.blockchain.ibm.com.cert")-->
  - [企业套餐的根 TLS 证书 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://blockchain-certs.mybluemix.net/3.secure.blockchain.ibm.com.rootcert)

  将内容保存到可在未来命令中引用这些内容的目录。

    ```
    mkdir tls-ibp
    cp us01.blockchain.ibm.com.cert $HOME/fabric-ca-client/tls-ibp/tls.pem
    ```
    {:codeblock}

6. 发出以下命令以同级管理员身份生成证书：

  ```
  fabric-ca-client enroll -u https://<enroll_id>:<enroll_password>@<ca_url_with_port> --caname <ca_name> --tls.certfiles <ca-tls_cert_file>
  ```
  {:codeblock}

  上面的 `<enroll_id>` 和 `<enroll_password>` 是已[使用网络监视器进行注册](#register-admin)的同级管理员的**标识**和**私钥**。`<ca_name>` 和 `<ca_url_with_port>` 是连接概要文件中 `caName` 和 `url` 的值。请去除 CA URL 开头的 `http://`。

  实际调用可能类似于以下示例命令：

  ```
  fabric-ca-client enroll -u https://peeradmin:peeradminpw@n7413e3b503174a58b112d30f3af55016-org1-ca.us3.blockchain.ibm.com:31011 --caname org1CA --tls.certfiles $HOME/fabric-ca-client/tls-ibp/tls.pem
  ```
  {:codeblock}

  此命令成功完成后，将在指定为 `$FABRIC_CA_CLIENT_HOME` 的目录中生成新的 MSP 文件夹。此目录包含将用于操作同级的证书。

7. 在刚刚创建的 MSP 文件夹中，打开新用户的 signCert 文件，并将其转换为 Base64 格式。您将在 MSP 目录的 `signcerts` 文件夹中找到该 signCert。如果使用的是示例步骤，那么可以使用以下命令：

  ```
  export FLAG=$(if [ "$(uname -s)" == "Linux" ]; then echo "-w 0"; else echo "-b 0"; fi)
  cat $HOME/fabric-ca-client/peer-admin/msp/signcerts/cert.pem | base64 $FLAG
  ```
  {:codeblock}

  **注：**使用以上命令生成的字符串的格式务必为一行。应该类似于以下内容：

   ```
   LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tDQpNSUlFbERDQ0EzeWdBd0lCQWdJUUFmMmo2MjdLZGNpSVE0dHlTOCs4a1RBTkJna3Foa2lHOXcwQkFRc0ZBREJoDQpNUXN3Q1FZRFZRUUdFd0pWVXpFVk1CTUdBMVVFQ2hNTVJHbG5hVU5sY25RZ1NXNWpNUmt3RndZRFZRUUxFeEIzDQpkM2N1WkdsbmFXTmxjblF1WTI5dE1TQXdIZ1lEVlFRREV4ZEVhV2RwUTJWeWRDQkhiRzlpWVd3Z1VtOXZkQ0JEDQpRVEFlRncweE16QXpNRGd4TWpBd01EQmFGdzB5TXpBek1EZ3hNakF3TURCYU1FMHhDekFKQmdOVkJBWVRBbFZUDQpNUlV3RXdZRFZRUUtFd3hFYVdkcFEyVnlkQ0JKYm1NeEp6QWxCZ05WQkFNVEhrUnBaMmxEWlhKMElGTklRVElnDQpVMlZqZFhKbElGTmxjblpsY2lC
   ```
   而不能如下所示：

   ```
   LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tDQpNSUlFbERDQ0EzeWdBd0lCQWdJUUFmMmo2MjdL
   ZGNpSVE0dHlTOCs4a1RBTkJna3Foa2lHOXcwQkFRc0ZBREJoDQpNUXN3Q1FZRFZRUUdFd0pWVXpF
   Vk1CTUdBMVVFQ2hNTVJHbG5hVU5sY25RZ1NXNWpNUmt3RndZRFZRUUxFeEIzDQpkM2N1WkdsbmFX
   VEFlRncweE16QXpNRGd4TWpBd01EQmFGdzB5TXpBek1EZ3hNakF3TURCYU1FMHhDekFKQmdOVkJB
   WVRBbFZUDQpNUlV3RXdZRFZRUUtFd3hFYVdkcFEyVnlkQ0JKYm1NeEp6QWxCZ05WQkFNVEhrUnBa
   ```

  此命令将显示类似于以下示例的字符串：

  ```
  LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNuRENDQWtPZ0F3SUJBZ0lVTXF5VDhUdnlwY3lYR2sxNXRRY3hxa1RpTG9Nd0NnWUlLb1pJemowRUF3SXcKYURFTTlEKaFhTTzRTWjJ2ZHBPL1NQZWtSRUNJQ3hjUmZVSWlkWHFYWGswUGN1OHF2aCtWSkhGeHBLUnQ3dStHZDMzalNSLwotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  ```

  在配置文件中 component 部分下的 `"admincerts"` 字段中输入此字符串。

### TLS CA 信息
{: #tls-ca-endpoint}

配置文件中的 `"tls"` 字段需要在 ICP 上部署的 CA 中的信息。您应将此 CA 用作单独的 TLS CA，这可使同级更安全。请使用以下指示信息来生成相关信息：

- `"cahost"` 和 `"caport"` 值是 [CA URL](CA_operate.html#ca-url) 的 URL 和端口。例如，如果 CA URL 为 `http://9.30.94.174:30167`，那么 `cahost` 的值将为 `9.30.94.174`，`caport` 的值将为 `30167`。
- `"caname"` 是在 ICP 上部署的 CA 的 TLS CA 名称。TLS CA 名称是在 CA 配置期间向 `CA TLS 实例名称`字段提供的值。
- `"cacert"` 是 CA 的 Base64 编码的 TLS 证书。作为先决条件，检索 [CA 的 TLS 证书](CA_operate.html#ca-tls)时，请使用命令输出的值来更新以下部分。

  ```
  "catls": {
    "cacert": ""
  ```
  {:codeblock}

  更新后，`"cacert"` 字段类似于以下示例：

  ```
  "catls": {
    "cacert": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUVsRENDQTN5Z0F3SUJBZ0lRQWYyajYyN0tkY2lJUTR0eVM4KzhrVEFOQmdrcWhraUc5dzBCQVFzRkFBkOHRiUWsKQ0FVdzdDMjlDNzlGdjFDNXFmUHJtQUVTcmNpSXhwZzBYNDBLUE1icDFaV1ZiZDQ9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0KCg=="
  ```

### 向 TLS CA 注册同级
{: #tls-register-peer}

您需要使用 Fabric CA 客户机向 ICP 上的 TLS CA 注册同级。

1. 现在，您在 `$HOME/fabric-ca-client/catls` 文件夹中应该有 TLS 证书文件 `tls.pem`。如果没有，您可以将[从 ICP 下载](CA_operate.html#ca-tls)的 TLS 证书复制到可以在下面命令中引用的目录。确保您位于 `$HOME/fabric-ca-client` 目录中。

  ```
  cd $HOME/fabric-ca-client
  mkdir catls
  cp $HOME/tls.pem $HOME/fabric-ca-client/catls/tls.pem
  ```
  {:codeblock}

2. 您需要使用 TLS CA 的管理员来进行注册。将 `$FABRIC_CA_CLIENT_HOME` 更改为要存储 TLS CA 管理员证书的目录。

  ```
  cd $HOME/fabric-ca-client
  mkdir tlsca-admin
  export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca-client/tlsca-admin
  ```
  {:codeblock}

3. 运行以下命令以 TLS CA 管理员身份生成证书。

  ```
  fabric-ca-client enroll -u https://<enroll_id>:<enroll_password>@<ca_url_with_port> --caname <tls_ca_name> --tls.certfiles <ca_tls_cert_file>
  ```
  {:codeblock}

  命令中的 `<enroll_id>` 和 `<enroll_password>` 是部署认证中心时传递到 Kubernetes 私钥的 [CA 管理员用户名和密码](CA_deploy_icp.html#admin-secret)。将 [CA URL](CA_operate.html#ca-url) 插入到 `<ca_url_with_port>` 中。请去除开头的 `http://`。`<tls_ca_name>` 是 [CA 配置](CA_deploy_icp.html#icp-ca-configuration-parms)期间指定的名称。

  `<ca_tls_cert_file>` 是包含完整路径的 [CA TLS 证书](CA_operate.html#ca-tls)文件名。

  实际调用可能类似于以下示例：

  ```
  fabric-ca-client enroll -u https://admin:adminpw@9.30.94.174:30167 --caname tlsca --tls.certfiles $HOME/fabric-ca-client/catls/tls.pem
  ```
  {:codeblock}

  您注册之后，就有了必需的证书来向 TLS CA 注册同级。

4. 发出以下命令查找亲缘关系和组织名称。

  ```
  fabric-ca-client affiliation list --caname <tls_ca_name> --tls.certfiles <tlsca_tls_cert_path>
  ```
  {:codeblock}

  命令可能类似于以下示例：

  ```
  fabric-ca-client affiliation list --caname tlsca --tls.certfiles $HOME/fabric-ca-client/catls/tls.pem
  ```

  您应该看到与以下示例类似的信息：

  ```
  affiliation: org1
      affiliation: org1.department1
  ```
  {:codeblock}

  记下您组织的**亲缘关系**值，例如以上示例中组织 `org1` 的 `org1.department1`。您需要在下面的命令中使用此值。

5. 运行以下命令以注册同级。

  ```
  fabric-ca-client register --caname <tls_ca_name> --id.name <name> --id.secret <secret>  --id.affiliation org1.department1 --id.type peer --tls.certfiles <tlsca_tls_cert_path>
  ```
  {:codeblock}

  在 `--caname` 字段中需要使用您的 TLS CA 名称。为同级创建名称和密码，然后将其用于替换 `name` 和 `secret`。**重要信息：**请记下这些信息。在配置文件的 `"tls"` 部分中，您需要输入 `name` 和 `secret` 作为 `"enrollid"` 和 `"enrollsecret"`。

  示例命令类似于以下内容：

  ```
  fabric-ca-client register --caname tlsca --id.affiliation org1.department1 --id.name peertls --id.secret peertlspw --id.type peer --tls.certfiles $HOME/fabric-ca-client/catls/tls.pem
  ```
  {:codeblock}

  发出此命令后，可以使用上面的值更新文件：

  ```
  "tls": {...
    },
    "enrollid": "peertls",
    "enrollsecret": "peertlspw",
  ```

  您只能注册一次身份。如果遇到问题，请尝试使用新的用户名和密码来执行命令。作为一项安全措施，每个身份以及附带的注册标识和私钥仅用于部署一个同级。请勿复用同级的标识和密码。

  命令成功完成后，应该会看到类似于以下示例的信息：

  ```
  2018/06/18 16:53:00 [INFO] Configuration file location: /fabric-ca-client/peer-admin/fabric-ca-client-config.yaml
  2018/06/18 16:53:00 [INFO] TLS Enabled
  2018/06/18 16:53:00 [INFO] TLS Enabled
  Password: peertlspw
  ```

可以运行 tree 命令来验证为准备配置文件所做的工作。浏览至存储证书的目录。tree 命令应该生成类似于以下结构的结果：

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
│       ├── cacerts
│       │   └── n4790a319014a473a8a65850c660396ab-org1-ca-us05-blockchain-ibm-com-31011-org1CA.pem
│       ├── keystore
│       │   └── 6c2a999ee80a6b0592c63c58f95193bea2df59efe5489938d513de47b83b372a_sk
│       ├── signcerts
│       │   └── cert.pem
│       └── user
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

### CSR（证书签名请求）主机
{: #csr-hosts}

您需要提供 CSR 主机名以部署同级。CSR 主机名包括将部署组件的集群的代理 IP 地址，以及将作为 Helm 图表主机名的`服务主机名`。

#### 找到集群代理 IP 地址的值

如果要在部署了 TLS CA 的同一 ICP 集群上部署同级，请输入[配置用于 TLS CA](CA_deploy_icp.html#icp-ca-configuration-parms) 时使用的代理 IP。如果要在其他集群上部署该组件，那么可以在 ICP 控制台中检索集群代理 IP 地址的值。您需要具有将部署同级的 ICP 集群的集群管理角色。

1. 登录到 ICP 控制台。在左侧导航面板中，单击**平台**，然后单击**节点**以查看在集群中定义的节点。
2. 单击具有角色`代理`的节点，然后从表中复制`主机 IP` 的值。
3. 在以下配置文件的 `"csr"` 部分中，将`主机 IP` 作为 `"hosts"` 的值插入：

  ```
  "csr": {
    "hosts": [""]
  }
  ```

#### 确定服务主机名

`服务主机名`是您在部署期间指定的 `Helm 发布名称`。如果集群 IP 代理地址为 9.42.134.44，并且 `Helm 发布名称`为 `org1peer1`，那么应在文件的 `"csr"` 部分中插入以下内容：

```
"csr": {
  "hosts": [
     "9.42.134.44",
     "org1peer1"
  ]
}
```
{:codeblock}

### 完成配置文件

完成所有步骤后，更新后的配置文件将类似于以下示例：

```
{
	"enrollment": {
		"component": {
			"cahost": "n5ec276985ff54d73b9bf0e0a6525d37b-org2-ca.stage.blockchain.ibm.com",
			"caport": "31011",
			"caname": "org1CA",
			"catls": {
				"cacert": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUVsRENDQTN5Z0F3SUJBZ0lRQWYyajYyN0tkY2lJUTR0eVM4KzhrVEFOQmdrcWhraUc5dzBCQVFzRkFBkOHRiUWsKQ0FVdzdDMjlDNzlGdjFDNXFmUHJtQUVTcmNpSXhwZzBYNDBLUE1icDFaV1ZiZDQ9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0KCg=="
			},
			"enrollid": "peer",
			"enrollsecret": "peerpw",
			"admincerts": ["LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNuRENDQWtPZ0F3SUJBZ0lVTXF5VDhUdnlwY3lYR2sxNXRRY3hxa1RpTG9Nd0NnWUlLb1pJemowRUF3SXcKYURFTTlEKaFhTTzRTWjJ2ZHBPL1NQZWtSRUNJQ3hjUmZVSWlkWHFYWGswUGN1OHF2aCtWSkhGeHBLUnQ3dStHZDMzalNSLwotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg=="]
		},
		"tls": {
			"cahost": "9.30.94.174",
			"caport": "30167",
			"caname": "tlsca",
			"catls": {
				"cacert": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUVsRENDQTN5Z0F3SUJBZ0lRQWYyajYyN0tkY2lJUTR0eVM4KzhrVEFOQmdrcWhraUc5dzBCQVFzRkFBkOHRiUWsKQ0FVdzdDMjlDNzlGdjFDNXFmUHJtQUVTcmNpSXhwZzBYNDBLUE1icDFaV1ZiZDQ9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0KCg=="
			},
			"enrollid": "peertls",
			"enrollsecret": "peertlspw",
			"csr": {
        "hosts": [
           "9.42.134.44",
           "org1peer1"
        ]
			}
		}
  }
}
```
{:codeblock}

填写完此文件后，您需要将其保存为 JSON 格式，并将其作为 Kubernetes 私钥传递到同级部署。使用[下一部分](#peer-config-file-ibp)中的步骤创建私钥。

## 创建配置私钥
{: #peer-config-file-ibp}

[Kubernetes Secret ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/configuration/secret/) 支持您保护和共享信息，而不必将其直接传递到部署。保存了配置文件后，需要对其进行 `base64` 编码。

1. 要将配置文件编码为 Base64 格式，请在终端窗口中运行以下命令：

  ```
  export FLAG=$(if [ "$(uname -s)" == "Linux" ]; then echo "-w 0"; else echo "-b 0"; fi)
  cat <config_file> | base64 $FLAG
  ```
  {:codeblock}

  **注：**使用以上命令生成的字符串的格式务必为一行。应该类似于以下内容：

   ```
   LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tDQpNSUlFbERDQ0EzeWdBd0lCQWdJUUFmMmo2MjdLZGNpSVE0dHlTOCs4a1RBTkJna3Foa2lHOXcwQkFRc0ZBREJoDQpNUXN3Q1FZRFZRUUdFd0pWVXpFVk1CTUdBMVVFQ2hNTVJHbG5hVU5sY25RZ1NXNWpNUmt3RndZRFZRUUxFeEIzDQpkM2N1WkdsbmFXTmxjblF1WTI5dE1TQXdIZ1lEVlFRREV4ZEVhV2RwUTJWeWRDQkhiRzlpWVd3Z1VtOXZkQ0JEDQpRVEFlRncweE16QXpNRGd4TWpBd01EQmFGdzB5TXpBek1EZ3hNakF3TURCYU1FMHhDekFKQmdOVkJBWVRBbFZUDQpNUlV3RXdZRFZRUUtFd3hFYVdkcFEyVnlkQ0JKYm1NeEp6QWxCZ05WQkFNVEhrUnBaMmxEWlhKMElGTklRVElnDQpVMlZqZFhKbElGTmxjblpsY2lC
   ```
   而不能如下所示：

   ```
   LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tDQpNSUlFbERDQ0EzeWdBd0lCQWdJUUFmMmo2MjdL
   ZGNpSVE0dHlTOCs4a1RBTkJna3Foa2lHOXcwQkFRc0ZBREJoDQpNUXN3Q1FZRFZRUUdFd0pWVXpF
   Vk1CTUdBMVVFQ2hNTVJHbG5hVU5sY25RZ1NXNWpNUmt3RndZRFZRUUxFeEIzDQpkM2N1WkdsbmFX
   VEFlRncweE16QXpNRGd4TWpBd01EQmFGdzB5TXpBek1EZ3hNakF3TURCYU1FMHhDekFKQmdOVkJB
   WVRBbFZUDQpNUlV3RXdZRFZRUUtFd3hFYVdkcFEyVnlkQ0JKYm1NeEp6QWxCZ05WQkFNVEhrUnBa
   ```

  保存生成的输出以供下面的步骤 4 使用。

2. 登录到 ICP 控制台。在左侧导航面板中，单击**配置**，然后单击**私钥**。单击**创建私钥**按钮以打开一个弹出面板，在其中允许您生成新的私钥对象。

3. 在**常规**选项卡上，填写以下字段：
  - **名称：**为私钥提供在集群中唯一的名称。部署同级时将使用此名称。名称必须全部为小写。  
  **注：**部署同级时，部署会自动生成名称为 `<helm_release_name>-secret` 的新私钥。因此，命名私钥时，请确保私钥的名称不同于 `<helm_release_name>-secret`。否则，Helm 图表部署会失败，因为它尝试创建的私钥已存在。
  - **名称空间：**要添加私钥的名称空间。选择要将同级部署到的`名称空间`。
  - **类型：**输入值`不透明`。

4. 单击此窗口的左侧导航窗格上的**数据**选项卡。在`名称`字段中，指定 `secret.json`，在值字段中，粘贴配置文件的 `base64` 编码的字符串。

5. （可选）如果计划使用 `CouchDB` 作为状态数据库，那么需要向此私钥对象再添加两个值。在**数据**选项卡中，单击**添加数据**按钮以添加部署容器时要用于数据库的 CouchDB 用户标识和密码。
  1. 创建用户名和密码，并将这些值编码为 Base64 格式。在终端窗口中运行以下命令，并将 `admin` 和 `adminpw` 替换为要使用的值。
    ```
    echo -n 'couch' | base64 $FLAG
    echo -n 'couchpw' | base64 $FLAG
    ```
    {:code_block}

   2. 在**名称**字段中，输入值 `couchdbuser`。在相应的**值**字段中，输入上面步骤 1 中的结果 `echo -n 'couch' | base64 $FLAG`。
   3. 单击**添加数据**按钮以添加第二个键值对。
   4. 在第二个**名称**字段中，输入值 `couchdbpwd`。在相应的**值**字段中，输入上面步骤 1 中的结果 `echo -n 'couchpw' | base64 $FLAG`。

6. 单击**创建**以保存私钥对象。

**注：**删除 Helm 发布时，不会从 ICP 集群中除去同级配置私钥。如果删除同级，那么还需要删除此私钥。

## 配置
{: #peer-configuration}

创建同级配置私钥对象后，可以使用以下步骤在 ICP 中配置和安装同级。一次只能安装一个同级。

1. 登录到 ICP 控制台，然后单击右上角的**目录**链接。
2. 单击左侧导航面板中的`区块链`，以找到标记为 `ibm-blockchain-platform-prod` 或 `ibm-blockchain-platform-dev`（如果已下载 Community Edition）的磁贴。单击磁贴以将其打开，您可以看到一个自述文件，其中包含有关安装和配置 Helm 图表的信息。
3. 单击面板顶部的**配置**选项卡，或单击右下角的**配置**按钮。
4. 指定[常规配置参数](#peer-global-parameters)的值并接受许可协议。
5. 打开`所有参数`折叠标记，并指定[全局配置参数](#peer-global-parameters)的值。
6. 向下滚动到**同级配置**部分。选中`安装同级`复选框，并完成同级的[配置设置](#peer-parameters)。
7. 单击**安装**。

### 配置参数
{: #ibp-peer-configuration-parms}

下表列出了 {{site.data.keyword.blockchainfull_notm}} Platform **特定于同级组件**的可配置参数及其缺省值。如果您是试用同级或者是第一次开始使用，请使用数据库和存储参数的缺省值。滚动到同级组件部分以选中`安装同级`复选框，并仅为该组件提供关联的配置信息。**尽管 Helm 图表 UI 指出不需要进一步配置，但您还是需要输入某些参数才能部署同级。**

#### 常规和全局配置参数
{: #peer-global-parameters}

|参数|描述|缺省值|必需|
| --------------|-----------------|-------|------- |
| `Helm 发布名称`|Helm 发布的名称。必须以小写字母开头并以任意字母数字字符结尾，并且只能包含连字符和小写字母数字字符。每次尝试安装组件时，都必须使用唯一的 Helm 发布名称。**重要信息：**此值必须与用于在 [JSON 私钥文件](#csr-hosts)中为“主机”字段生成“服务主机名”的值相匹配。|无|是|
|`目标名称空间`|选择 Kubernetes 名称空间以安装 Helm 图表。|无|是|
|**全局配置**|应用于 Helm 图表中所有组件的参数。|||
|`服务帐户名称`|输入将用于运行 pod 的[服务帐户 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)。|default|否|

#### 同级配置参数
{: #peer-parameters}

|参数|描述|缺省值|必需|
| --------------|-----------------|-------|------- |
|**集群配置**|**集群配置信息**| ||
|`安装同级`|选中以安装同级。|未选中|是，如果要安装同级|
|`同级工作程序节点体系结构`|选择云平台体系结构（AMD64 或 S390x）。|AMD64|是|
|`同级映像存储库`|同级 Helm 图表的位置。此字段会自动填充为安装的路径。如果您使用的是 Community Edition，但无法访问因特网，那么此值应该与 Fabric 同级映像的下载目录相匹配。|ibmcom/ibp-fabric-peer|是|
|`同级 Docker 映像标记`|与同级映像关联的标记的值。|1.2.1，自动填充为正确的值。|是|
|`同级配置`|可以通过在此字段中粘贴您自己的 `core.yaml` 配置文件来定制同级配置。要查看样本 `core.yaml` 文件，请参阅 [`core.yaml` 样本配置 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://github.com/hyperledger/fabric/blob/release-1.2/sampleconfig/core.yaml)（**仅限高级用户**）。|无|否|
|`同级配置私钥（必需）`|在 ICP 中创建的[同级配置私钥](#peer-config-secret)的名称。|无|是|
|`组织 MSP（必需）`|在“网络监视器”(IBP UI) 中，可以通过单击“概述”屏幕上的“远程同级配置”找到此值。|无|是|
|`同级服务类型`|用于指定是否应该在同级上[公开外部端口 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)。选择 NodePort 以向外部公开端口（建议），选择 ClusterIP 以不公开端口。此发行版中不支持 LoadBalancer 和 ExternalName。|NodePort|是|
|`状态数据库`|用于存储通道分类帐的[状态数据库](../glossary.html#state-database)。同级需要使用与[区块链网络](../v10_dashboard.html#network-preferences)相同的数据库。|LevelDB|是|
|`CouchDB 映像存储库`|仅当选择了 CouchDB 作为分类帐数据库时才适用。此字段会自动填充为安装的路径。如果您使用的是 Community Edition，但无法访问因特网，那么此值应该与 Fabric CouchDB 映像的下载目录相匹配。|ibmcom/ibp-fabric-couchdb|是|
|`CouchDB Docker 映像标记`|仅当选择了 CouchDB 作为分类帐数据库时才适用。与 CouchDB 映像关联的标记的值。|自动填充为正确的值。|是|
|`同级数据持久性已启用`|启用在集群重新启动或发生故障后持久存储数据的功能。有关更多信息，请参阅 [Kubernetes 中的存储器 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/storage/ "卷")。*如果未选中此项，那么在故障转移或 pod 重新启动时将丢失所有数据。*|已选中|否|
|`同级使用动态供应`|选中此项以对存储卷启用动态供应。|已选中|否|
|`同级持久卷声明`|仅限新的声明。输入要创建的新持久卷声明 (PVC) 的名称。|my-data-pvc|否|
|`同级存储类名称`|指定同级的存储类名称。|如果创建新的 PVC，那么为空白；否则，请指定与现有 PVC 关联的存储类。|否|
|`同级现有卷声明`|指定现有卷声明的名称，并使其他所有字段保留空白。|无|否|
|`同级选择器标签`|PVC 的[选择器标签 ![外部链接图标](../images/external_link.svg "外部链接图标") ](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/ "标签和选择器")。|无|否|
|`同级选择器值`|PVC 的[选择器值 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/ "标签和选择器")。|无|否|
|`同级存储器访问方式`|指定用于 PVC 的存储器[访问方式 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes "访问方式")。|ReadWriteMany|否|
|`同级卷声明大小`|卷声明的大小必须大于 2 Gi。|8 Gi|是|
|`状态数据库持久卷声明`|仅限新的声明。输入要创建的新持久卷声明 (PVC) 的名称。|statedb-pvc|否|
|`状态数据库存储类名称`|指定状态数据库的存储类名称。|无|否|
|`状态数据库现有卷声明`|指定现有卷声明的名称，并使其他所有字段保留空白。|无|否|
|`状态数据库选择器标签`|PVC 的[选择器标签](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)。|无|否|
|`状态数据库选择器值`|PVC 的[选择器值](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)。|无|否|
|`状态数据库存储器访问方式`|指定用于 PVC 的存储器[访问方式 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes "访问方式")。|ReadWriteMany|否|
|`状态数据库卷声明大小`|选择要使用的磁盘的大小。|8 Gi|是|
|`CouchDB - 数据持久性已启用`|对于 CouchDB 容器，分类帐数据将在容器重新启动后可用。*如果未选中此项，那么在故障转移或 pod 重新启动时将丢失所有数据。*|已选中|否|
|`CouchDB - 使用动态供应`|对于 CouchDB 容器，请使用 Kubernetes 动态存储器。|已选中|否|
|`同级 CPU 请求`|分配给同级的最小 CPU 数。|1|是|
|`同级 CPU 限制`|分配给同级的最大 CPU 数。|2|是|
|`同级内存请求`|分配给同级的最小内存量。|1 Gi|是|
|`同级内存限制`|分配给同级的最大内存量。|4 Gi|是|
|`CouchDB CPU 请求`|分配给 CouchDB 的最小 CPU 数。|1|是|
|`CouchDB CPU 限制`|分配给 CouchDB 的最大 CPU 数。|2|是|
|`CouchDB 内存请求`|分配给 CouchDB 的最小内存量。|1 Gi|是|
|`CouchDB 内存限制`|分配给 CouchDB 的最大内存量。|4 Gi|是|


选择了 CouchDB 作为同级数据库时，将在 pod 中创建两个容器，一个容器用于同级，另一个容器用于 CouchDB。同级容器包含同级 PVC 的单个卷安装，用于在文件系统上存储区块和事务。CouchDB 容器会安装包含分类帐数据的同级状态数据库 PVC。

<!-- LevelDB is not supported on the Peer
When LevelDB is selected as the peer database, a single container is created in the pod for running both the peer and LevelDB
processes. This container has two volume mounts, one for the Peer PVC and the second volume mount is for the peer state database PVC that contains the ledger data.
-->
<!--
| Peer database selection  | Contents of Container #1  | Contents of Container #2 |
| --------------|-----------------|---------------|
| CouchDB | Peer that mounts the Peer PVC| CouchDB that mounts the state database PVC |
| LevelDB | Peer and LevelDB that mount the Peer PVC and the state database PVC | n/a |
-->

### 使用 Helm 命令行安装 Helm 发布
{: #ibp-helm-cli}

或者，可以使用 Helm CLI 来安装 Helm 发布。运行 `helm install` 命令之前，请确保[将集群的 Helm 存储库添加到 Helm CLI 环境 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/app_center/add_int_helm_repo_to_cli.html "将内部 Helm 存储库添加到 Helm CLI")。

您可以通过创建 `yaml` 文件并将其传递到以下 `helm install` 命令来设置安装所需的参数。

```
helm install --name <helm_release_name>  <helm_chart> \
--version <helm_chart_version> \
--values <customvalues.yaml> \
--tls
```

其中：
- `<helm_release name>` 表示您要为 Helm 发布提供的名称。
- `<helm_chart>` 表示导入到目录中的 Helm 图表的名称。
- `<helm_chart_version>` 表示导入到目录中的 Helm 图表的版本。
- `<customvalues.yaml>` 是包含配置参数的 yaml 文件的名称。

例如：

```
helm install --name jnchart2 mycluster/ibm-blockchain-platform \
--version 1.1.0 \
--values peer-s390x-values.yaml \
--tls
```
您可以通过编辑已下载归档文件中包含的 `values.yaml` 来创建新的 `yaml` 文件，values.yaml 文件包含所有必需参数及其缺省设置。

## 验证同级安装
{: #verify-installation-ibp}

完成配置参数并单击**安装**按钮后，单击**查看 Helm 发布**按钮可查看部署。如果部署成功，您应该会在“部署”表中看到 `DESIRED`、`CURRENT`、`UP TO DATE` 和 `AVAILABLE` 字段中的值为 1。您可能需要单击“刷新”并等待该表更新。您还可以通过单击 ICP 控制台左上角的**菜单**图标来找到“部署”表。在菜单列表中，单击**工作负载**，然后单击 **Helm 发布**。

如果向下滚动到`注释`部分，可看到在[操作同级](peer_operate_ibp.html)时将使用的重要信息。

<!--
### Verifying the peer can connect to Starter or Enterprise Plan network

You can run a peer CLI command from inside the peer container to verify that your peer has connected to your network on the {{site.data.keyword.blockchainfull_notm}} Platform. Complete the following instructions to run the `peer channel fetch` command to fetch the genesis block from a channel:

1. If you have not already connected to your ICP cluster, follow these [instructions](peer_operate_ibp.html#peer-kubectl-configure) to connect to it and use the cli from inside the peer container.

2. If you deploy your peer behind a firewall, you need to open a passthru to enable the network on the platform to complete a TlS handshake with your peer. You only need to enable a passthru for the Node port bound to port 7051 of your peer. For more information, see [finding your peer endpoint information](peer_operate_ibp.html#peer-endpoint).

3. Your organization needs to be added to a channel in the network before you can fetch the genesis block.

  - You can start a new channel for the peer. If you are using IBP Starter or Enterprise Plan, you can automatically include your organization during [channel creation](create_channel.html#creating-a-channel).

  - Another member of the blockchain network can also add your organization to an existing channel by using a [channel update](create_channel.html#updating-a-channel).

    The peer uploads its signCert during installation, so that you need to only synchronize the certificate to the channel. On the "Channels" screen of the Network Monitor, locate the channel that your organization joined and select **Sync Certificate** from the drop-down list under the **Action** header. This action synchronizes the certificates across all the peers on the channel.

4. You also need to upload the signCert of your peer admin to Starter Plan or Enterprise Plan in order for a remote CLI or application to perform channel operations such as fetching the channel genesis block and instantiating chaincode.
    1. On your local machine, cat the file `HOME/fabric-ca-client/peer-admin/msp/signcerts/cert.pem` and then copy it to the clipboard.
    2. Enter the Network Monitor of your network on {{site.data.keyword.blockchainfull_notm}} Platform.
    3. Click **Members** in the left navigator and click the **Certificates** tab.
    4. Click the **Add Certificate** button.
    5. In the **Add certificate** window, give your certificate a name, such as "peer-admin", paste in the certificate you just copied to the clipboard and click **Submit**.
    6. You will be asked whether you want to restart the peers. Click **Restart**.
    7. Click **Channels** in the left navigator and locate the channel that the peer will join.
    8. Click the three dots under the **Actions** header and click **Sync Certificate**. In the **Sync certificate** window, click **Submit**.

5. Retrieve configuration information from your Connection profile available on the "Overview" screen of your Network Monitor. Click **Connection Profile** and then **Download**.

  - Find the orderer URL by searching for **orderers**, which is located under `orderers > url`. Make a note of the URL that begins with the network name. The URL is similar to the following example:

    ```
    ash-zbc07b.4.secure.blockchain.ibm.com:21239
    ```

  - Find the name of your organization by searching for **organizations**. This should be the same organization as you use to register your peer. You can find your organization's name together with its associated `mspid`. Make a note of the value of the `mspid`.

6. Run the following commands to set the environment variables in the peer container.

  ```
  export ORDERER_1=<ORDERER_URL>
  export CHANNEL=<CHANNEL_NAME>
  export CC_NAME=<CC_NAME>
  export ORGID=<ORGANIZATION_MSP_ID>
  export PEERADDR=<PEER_ADDR>
  ```
  {:codeblock}

  Replace the fields with your own information.
  - Replace `<ORDERER_URL>` with the hostname and port of the orderer from the `creds.json` file.
  - Replace `<CHANNEL_NAME>` with the name of the channel that the peer joins.
  - Replace `<CC_NAME>` with any name to refer to your chaincode.
  - Replace `<ORGANIZATION_MSP_ID>` with the name of the organization from the `creds.json` file.
  - Replace `<PEER_ADDR>` with `localhost:7051`

  For example:

  ```
  export ORDERER_1=ash-zbc07b.4.secure.blockchain.ibm.com:21239
  export CHANNEL=defaultchannel
  export CC_NAME=mycc
  export ORGID=org1
  export PEERADDR=localhost:7051
  ```
  {:codeblock}

7. Run the following peer CLI command to fetch the genesis block of the channel.

  ```
  CORE_PEER_TLS_ROOTCERT_FILE=/mnt/certs/tls/cacert.pem CORE_PEER_TLS_ENABLED=true CORE_PEER_ADDRESS=${PEERADDR} CORE_PEER_LOCALMSPID=${ORGID} CORE_PEER_MSPCONFIGPATH=/mnt/crypto/peer/peer/msp/ GOPATH=/ peer channel fetch 0 -o ${ORDERER_1} -c ${CHANNEL} --cafile /mnt/certs/tls/cacert.pem --tls
  ```
  {:codeblock}

  **Note:** It is possible that when you run any of these CLI commands, you might experience the following warning that can be safely ignored.

  ```
  [msp] getPemMaterialFromDir -> WARN 001 Failed reading file
  /mnt/crypto/peer/peer/msp/intermediatecerts/<intermediate cert name>.pem: no pem content for file /mnt/crypto/peer/peer/msp/intermediatecerts/<intermediate cert name>.pem
  ```

  Verify that the command worked successfully and that genesis block is added to your peer container by running the following command:

  ```
  ls *.block
  ```
  {:codeblock}

  You know that the genesis block is added successfully when you see something that is similar to the following example:

  ```
  defaultchannel_0.block
  ```

  Successfully fetching the genesis block indicates that your peer can connect to your Starter or Enterprise Plan network.
  You still need to join the peer to the channel and install chaincode. See [operating your peer](peer_operate_ibp.html) for more information.

-->

## 查看同级日志
{: #peer-ibp-view-logs}

在命令行中使用 [`kubectl CLI 命令`](peer_operate_ibp.html#peer-kubectl-configure)或通过 ICP 集群中包含的 [Kibana ![外部链接图标](../images/external_link.svg "外部链接图")](https://www.elastic.co/products/kibana "您使用 Elasticsearch 的窗口")，可以查看组件日志。有关更多信息，请参阅[有关访问日志的指示信息](peer_operate_ibp.html#peer-ibp-view-logs)。

## 下一步做什么

部署了同级后，需要完成若干操作步骤，然后才能将交易提交到区块链网络的分布式分类账，并读取该分布式分类账。有关更多信息，请参阅[在使用入门套餐或企业套餐的环境中操作同级](peer_operate_ibp.html)。
