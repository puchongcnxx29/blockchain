---

copyright:
  years: 2018
lastupdated: "2018-12-07"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}

# 在 {{site.data.keyword.cloud_notm}} Private 上操作认证中心
{: #ca-operate}

***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

认证中心 (CA) 用于在网络上提供身份。CA 可被视为类似于公共可信公证方，用于在多个参与方之间建立信任。将为网络中的每个实体提供一个根 CA 对其签名的证书，以封装相应实体的数字身份。此证书是在网络上执行的所有签名和验证操作的信任根。
{:shortdesc}

多云区块链网络中的每个组织都应该部署自己的 CA。您组织的 CA 将对组织所操作的组件（例如，排序服务、同级或应用程序）的请求进行签名。因此，在部署其他任何组件之前，您需要先部署 CA。

您需要执行一些必备步骤才能操作认证中心：

- [配置 CLI](#ca-kubectl-configure)
- [检索认证中心 URL](#ca-url)
- [下载 CA TLS 证书](#ca-tls)
- [设置 Fabric CA 客户机](#fabric-ca-client)
- [使用 CA 管理员生成证书](#enroll-admin)

完成必备步骤后，可以使用 CA 在组织的网络中注册新身份，并生成应用程序可以使用的证书。

- [使用 CA 部署排序节点或同级](#deploy-orderer-peer)
<!-- [Register and enroll client applications](#register-enroll-app)-->

本指南还提供了有关操作 CA 以及如何使用其证书来管理区块链网络的一些重要概念信息。

- [成员资格服务提供者 (MSP)](#msp)
- [TLS 证书](#tls)

## 先决条件
{: #prerequisites}

无论您是创建还是加入网络，都需要完成这些必备步骤，才能使用 CA 并部署属于您组织的其他组件和身份。

### 配置 CLI
{: #ca-kubectl-configure}

您需要使用 **kubectl** 命令行工具来连接到在 ICP 中运行的 CA 容器。

1. 登录到 ICP 集群 UI。浏览至**命令行工具**选项卡，然后单击 **Cloud Private CLI**。您将看到以下可以下载的工具。

   * 安装 IBM Cloud Private CLI 和插件
   * 安装 Kubectl CLI
   * 安装 Helm
   * 安装 Istio CLI

  要操作 CA，您需要使用前**三个**工具，其中 Helm 是可选的。单击每个工具，然后针对您使用的机器类型，运行相应的 `curl` 命令。接下来，对每个工具发出 `chmod` 和 `sudo mv` 命令。`chmod` 命令将更改相关 CLI 的许可权以使其可执行，`sudo mv` 命令将移动文件并对其重命名。

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

  如果该命令成功，将返回类似于以下示例的响应，其中 `ca-6cf85f6687-hcxpl` 是 CA 容器的名称。

  ```
  NAME                                 READY      STATUS             RESTARTS   AGE
  ca-6cf85f6687-hcxpl          2/2        Running            0          6h
  ```

  您现在已准备就绪，可以使用 **kubectl** 工具来获取 CA URL。

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

### 检索认证中心 URL
{: #ca-url}

您需要将 CA URL 作为目标，以便请求生成证书或注册新身份。可以使用 ICP 控制台 UI 来查找 CA URL。您需要是[集群管理员 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/user_management/assign_role.html "集群管理员角色和操作") 才能完成以下步骤：

1. 登录到 ICP 控制台，然后单击左上角的**菜单**图标。
2. 单击**工作负载** > **Helm 发布**。
3. 找到您的 Helm 发布的名称，然后打开 Helm 发布详细信息面板。
4. 向下滚动到面板底部的**注释**部分。在**注释**部分中，您会看到一组命令，可帮助您操作 CA 部署。
5. 如果尚未配置与 CA 容器进行交互所需使用的 [kubectl CLI](#ca-kubectl-configure)，请执行相关步骤进行配置。
6. 在 CLI 中，运行注释中的第一个命令，此命令位于 **1. 通过运行以下命令获取应用程序 URL：**之后。此命令将显示 CA URL 和端口，类似于以下示例。将此值保存为应用程序 URL 以用于后续命令。

  ```
  http://9.30.94.174:30167
  ```

**注：**如果将 CA 部署在防火墙后，那么需要打开传递，以支持平台上的网络与 CA 完成 TLS 握手。您只需要对与 CA URL 并置在一起的端口启用传递。


### 检索 CA TLS 证书
{: #ca-tls}

您需要下载 CA TLS 证书，并随命令一起传递，才能通过远程客户机与 CV 进行通信。

1. 登录到 ICP 控制台。单击左上角的**菜单**图标。
2. 单击**工作负载** > **Helm 发布**。
3. 找到您的 Helm 发布的名称，然后打开 Helm 发布详细信息面板。
4. 向下滚动到面板底部的**注释**部分。在**注释**部分中，您会看到一组命令，可帮助您操作 CA 部署。
5. 如果尚未配置与 CA 容器进行交互所需使用的 [kubectl CLI](#ca-kubectl-configure)，请执行相关步骤进行配置。
6. 在 CLI 中，运行注释中的第三个命令，此命令位于 **3. 获取 TLS 证书：**之后。此命令将 TLS 证书保存为本地目录上的 `tls.pem` 文件。在未来的命令中需要引用此证书。请记下其位置。
7. 此命令还会将证书转换为 Base64 格式并显示该证书。结果类似于以下字符串：

  ```
    LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUVsRENDQTN5Z0F3SUJBZ0lRQWYyajYyN0tkY2lJUTR0eVM4KzhrVEFOQmdrcWhraUc5dzBCQVFzRkFBkOHRiUWsKQ0FVdzdDMjlDNzlGdjFDNXFmUHJtQUVTcmNpSXhwZzBYNDBLUE1icDFaV1ZiZDQ9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0KCg==
  ```

  **重要信息：**您需要复制此字符串，并将其保存为 `ICP CA TLS 证书`。部署其他组件时将使用此证书。

### 设置 Fabric CA 客户机
{: #fabric-ca-client}

您可以使用 Fabric CA 客户机来操作 CA。以下指示信息说明如何使用 Fabric CA 客户机来登记和注册属于您组织的其他组件的身份。您还可以使用 Fabric SDK 来完成必备步骤。

1. 您需要将 [Fabric CA 客户机 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#fabric-ca-client "下载 Fabric CA 客户机") 下载到本地文件系统。

  获取 Fabric CA 客户机的最简单方法是直接下载所有 Fabric 工具二进制文件。浏览至要在其中使用命令行下载二进制文件的目录，然后通过运行以下 [Curl ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/prereqs.html#install-curl "Curl") 命令来访存这些二进制文件。

  ```
  curl -sSL http://bit.ly/2ysbOFE | bash -s 1.2.1 1.2.1 -d -s
  ```
  {:codeblock}

  此命令将在 `bin/` 目录中安装二进制文件。

2. 将 `PATH` 路径设置为 Fabric 工具二进制文件的下载目录：

  ```
  export PATH=$PATH:<full/path/to/fabric-client/bin>
  ```
  {:codeblock}

  例如，如果已将二进制文件安装在主目录中，那么应将 `PATH` 设置为：

  ```
  export PATH=$PATH:$HOME/bin
  ```
  {:codeblock}

3. 创建用于存储 CA 管理员证书的文件夹。

  ```
  mkdir -p $HOME/fabric-ca-client/ca-admin
  ```
  {:codeblock}

4. 将 `$FABRIC_CA_CLIENT_HOME` 环境变量的值设置为 CA 客户机将用于存储所生成 [MSP](#msp) 证书的路径。请确保除去早先可能尝试创建的配置资料。如果之前未运行过 `enroll` 命令，那么 `msp` 文件夹和 `.yaml` 文件不存在。

  ```
  export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca-client/ca-admin
  rm -rf $FABRIC_CA_CLIENT_HOME/fabric-ca-client-config.yaml
  rm -rf $FABRIC_CA_CLIENT_HOME/msp
  ```
  {:codeblock}

5. 将[从 ICP 下载](#ca-tls)的 TLS 证书复制到可以在未来命令中引用该证书的目录。确保您位于 `$HOME/fabric-ca-client` 目录中。

  ```
  cd $HOME/fabric-ca-client
  mkdir catls
  cp $HOME/tls.pem $HOME/fabric-ca-client/catls/tls.pem
  ```
  {:codeblock}

### 使用 CA 管理员生成证书
{: #enroll-ca-admin}

使用 CA 来部署组件或客户机应用程序之前，需要生成证书以将您认证为能够注册新用户的管理员。生成必需的证书、专用密钥和公用证书（也称为注册证书或 signCert）的过程被称为**注册**。

只能使用已向认证中心注册的身份来生成证书。提供身份的名称和私钥以生成证书。部署 CA 时，会自动为您注册 **CA 管理员**身份。现在，您可以使用该管理员名称和密码向 Fabric CA 客户机发出 `enroll` 命令，以生成包含证书的 MSP 文件夹，然后这些证书可用于注册其他同级或排序节点身份。

1. 确保完成[设置 Fabric CA 客户机](#fabric-ca-client)的步骤，并确保将 `$FABRIC_CA_CLIENT_HOME` 设置为要用于存储 CA 管理员证书的目录。

  ```
  echo $FABRIC_CA_CLIENT_HOME
  $HOME/fabric-ca-client/ca-admin
  ```
  {:codeblock}

2. 运行以下命令生成证书：

  ```
  fabric-ca-client enroll -u https://<enroll_id>:<enroll_password>@<ca_url_with_port> --caname <ca_name>  --tls.certfiles  <ca_tls_cert_path>
  ```
  {:codeblock}

  命令中的 `<enroll_id>` 和 `<enroll_password>` 是部署认证中心时传递到 Kubernetes 私钥的 [CA 管理员用户名和密码](CA_deploy_icp.html#admin-secret)。将 [CA URL](#ca-url) 插入到 `<ca_url_with_port>` 中。请去除开头的 `http://`。`<ca_name>` 是在部署 CA 时向 `CA 名称`字段提供的值。

  `<ca_tls_cert_path>` 是 [CA TLS 证书](#ca-tls)的完整路径。

  实际调用可能类似于以下示例命令：

  ```
  fabric-ca-client enroll -u https://admin:adminpw@9.30.94.174:30167 --caname org1CA --tls.certfiles $HOME/fabric-ca-client/catls/tls.pem
  ```

  此 `enroll` 命令会生成完整的证书集，称为成员资格服务提供者 (MSP) 文件夹，该文件夹位于设置为 Fabric CA 客户机的 `$HOME` 路径的目录中。例如，`$HOME/fabric-ca-client/ca-admin`。有关 MSP 以及 MSP 文件夹所包含内容的更多信息，请参阅[成员资格服务提供者](#msp)。

  如果此 `enroll` 命令失败，请参阅[故障诊断主题](#ca-enroll-error)以了解可能的原因。

您可以运行 tree 命令来验证是否已完成所有必备步骤。浏览至存储证书的目录。tree 命令应该生成类似于以下结构的结果：

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
└── catls
    └── tls.pem    
```

## 使用 CA 部署排序节点或同级
{: #deploy-orderer-peer}

部署排序节点或同级之前，您需要创建配置 JSON 文件，其中包含有关组件身份和您的 CA 的重要信息。然后，您需要在配置期间使用 [Kubernetes Secret ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/configuration/secret/) 对象将此文件传递到 Helm 图表。通过此文件，排序节点或同级可从 CA 获取加入区块链网络所需的证书。此文件还包含管理员证书，用于允许您以管理员用户身份运行组件。

以下指示信息提供了[模板 JSON 配置文件](#config-file-template)（您可以编辑该模板并保存到本地文件系统），并指导您如何使用 CA 来填写此文件。

- 如果要在 ICP 上部署排序节点或部署同级以连接到在 ICP 上托管的联盟，请遵循以下指示信息进行操作。
- 如果要部署同级以连接到入门套餐或企业套餐，请遵循[在 IBM Cloud Private 中部署同级以连接到入门套餐或企业套餐](/docs/services/blockchain/peer_deploy_ibp.html)上的指示信息进行操作。这些步骤将指导您如何在 {{site.data.keyword.blockchainfull_notm}} Platform 上使用 CA 在 ICP 上部署同级。

### 配置文件
{: #config-file-template}

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

将这整个文件复制到文本编辑器中，您可以在其中编辑该文件并将其作为 JSON 文件保存到本地文件系统中。

### CA 连接信息

首先，我们需要提供 ICP 上 CA 的连接信息。使用以下信息来填写文件的 `"component"` 部分中的值：

- `"cahost"` 和 `"caport"` 值是 [CA URL](#ca-url) 中的 URL 和端口。例如，如果 CA URL 为 http://9.30.94.174:30167，那么 `"cahost"` 的值将为 `9.30.94.174`，`"caport"` 的值将为 `30167`。
- `"caname"` 是部署 CA Helm 图表时指定的 CA 名称。使用 [CA 管理员身份](#enroll-admin)注册时，在命令中引用了该 CA 名称。
- `"cacert"` 是 CA 的 Base64 编码的 TLS 证书。通过使用 Helm 发布中的注释，可在 ICP 控制台中[检索 CA 的 TLS 证书](#ca-tls)，然后将 note 命令的 Base64 格式输出插入到以下部分中。
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

- 以下 `"tls"` 部分中的字段需要您传递到上面 component 部分的相同信息，但您需要使用 CA 部署期间指定的 CA TLS 实例名称的值。对 `"cacert"` 值使用相同的 TLS 证书。
  ```
  "tls": {
    "cahost": "",
    "caport": "",
    "caname": "",
    "catls": {
      "cacert": ""
  ```
  {:codeblock}

检索到认证中心连接信息后，需要使用 Fabric CA 客户机来完成若干操作步骤：

1. [向 CA 注册新组件](#register-component)。

2. 您还需要[注册组件管理员](#register-admin)，然后在 MSP 文件夹中[为组件管理员生成证书](#enroll-admin)。如果您已经注册了管理员来部署其他组件，那么无需完成此步骤。

3. [向 TLS CA 注册新组件](#tls-register-component)。

### 向 CA 注册组件身份
{: #register-component}

如果要通过部署排序服务并向其添加组织来建立联盟，或者要部署同级并将其加入通道，那么您首先需要向 CA 注册组件身份。然后，组件部署可以生成同级或排序节点加入网络所必需的证书。

1. 通过 Fabric CA 客户机[使用 CA 管理员生成证书](#enroll-ca-admin)。使用这些管理员证书发出以下命令。确保 `$FABRIC_CA_CLIENT_HOME` 设置为 `$HOME/fabric-ca-client/ca-admin`。

  ```
  echo $FABRIC_CA_CLIENT_HOME
  $HOME/fabric-ca-client/ca-admin
  ```

2. 发出以下命令查找亲缘关系和组织名称。
  ```
  fabric-ca-client affiliation list --caname <ca_name> --tls.certfiles <ca_tls_cert_path>
  ```
  {:codeblock}

  命令可能类似于以下示例：
  ```
  fabric-ca-client affiliation list --caname org1CA --tls.certfiles $HOME/fabric-ca-client/catls/tls.pem
  ```

  您应该看到与以下示例类似的信息：
  ```
  affiliation: org1
      affiliation: org1.department1
  ```

  记下第二个 **affiliation** 值，例如，`org1.department1`。您需要在下面的命令中使用此值。

3. 运行以下命令注册排序节点或同级。

  ```
  fabric-ca-client register --caname <ca_name> --id.name <name> --id.affiliation org1.department1 --id.type <component_type> --id.secret <secret> --tls.certfiles <ca_tls_cert_path>
  ```
  {:codeblock}

  为组件创建名称和密码，然后将其用于替换 `name` 和 `secret`。**重要信息：**请记下这些信息。如果要部署排序节点，请将 `--id.type` 设置为 `orderer`，如果要部署同级，请将其设置为 `peer`。该命令可能类似于以下示例：

  ```
  fabric-ca-client register --caname org1CA --id.affiliation org1.department1 --id.name peer --id.secret peerpw --id.type peer --tls.certfiles $HOME/fabric-ca-client/catls/tls.pem
  ```

  运行此命令后，需要在配置文件的 `"component"` 部分下，输入 `name` 和 `secret` 作为 `"enrollid"` 和 `"enrollsecret"`。

  ```
  "component": {...
    },
    "enrollid": "peer1",
    "enrollsecret": "peerpw",
  ```

  您只能注册一次身份。如果遇到问题，请尝试使用新的用户名和密码来执行命令。作为一项安全措施，每个身份以及附带的注册标识和私钥仅用于部署一个同级。虽然可以将一个**管理员**身份用于多个组件（这是建议的部署策略），但不要复用同级标识和密码。

  命令成功完成后，应该会看到类似于以下示例的信息：

  ```
  2018/06/18 16:53:00 [INFO] Configuration file location: /fabric-ca-platform/admin/fabric-ca-client-config.yaml
  2018/06/18 16:53:00 [INFO] TLS Enabled
  2018/06/18 16:53:00 [INFO] TLS Enabled
  Password: peerpw
  ```

#### 注册管理员
{: #register-admin}

注册组件之后，还需要创建可用于操作组件的管理员身份。首先，您需要向 CA 注册此新身份，并将其用于生成 MSP 文件夹。然后，需要将管理员用户的 signCert 添加到配置文件，在部署期间此证书将成为排序节点或同级的管理员证书。这允许您使用管理员身份的证书来操作区块链网络，例如在同级上启动新的通道或安装链代码。

您只需要针对属于您组织的组件创建一个管理员身份。如果要部署多个同级或一个排序节点，那么只需要完成这些步骤一次。您可以使用为一个组件生成的 signCert 来部署任何同级或排序节点。

确保将 `$FABRIC_CA_CLIENT_HOME` 设置为 CA 管理员的 MSP 的路径。

```
echo $FABRIC_CA_CLIENT_HOME
$HOME/fabric-ca-client/ca-admin
```
{:codeblock}

通过运行以下命令向 CA 注册组件管理员身份：

```
fabric-ca-client register --caname <ca_name> --id.name <name> --id.affiliation org1.department1 --id.type user --id.secret <password> --tls.certfiles <ca_tls_cert_path>
```
{:codeblock}

为管理员创建新的用户身份 `name` 和 `secret`。此身份将成为您部署的所有组件的管理员。请确保使用的值与您刚刚注册的同级或排序节点身份的值不同。该命令类似于以下示例。

```
fabric-ca-client register --caname org1CA --id.name peeradmin --id.affiliation org1.department1 --id.type user --id.secret peeradminpw --tls.certfiles $HOME/fabric-ca-client/catls/tls.pem
```

**重要信息：**请记下这些信息。您将使用此 `name` 和 `secret` 通过 `enroll` 命令来生成 MSP 文件夹。

#### 生成管理员的 MSP 文件夹
{: #enroll-admin}

注册组件管理员之后，可以使用以下命令生成证书：

```
fabric-ca-client enroll -u https://<peer_admin_enroll_id>:<peer_admin_enroll_password>@<ca_url_with_port> --caname <ca_name> -M $HOME/path/to/peer-admin/msp --tls.certfiles <ca_tls_cert_path>
```
{:codeblock}

例如：

```
fabric-ca-client enroll -u https://peeradmin:peeradminpw@9.30.94.174:30167 --caname org1CA -M $HOME/fabric-ca-client/peer-admin/msp --tls.certfiles $HOME/fabric-ca-client/catls/tls.pem
```

此命令成功完成后，将在使用 `-M` 标志指定的目录中生成新的 MSP 文件夹。此目录包含将用于操作组件的证书。可以使用 tree 命令来验证 enroll 命令是否有效。浏览至存储证书的目录。tree 命令应该生成类似于以下结构的结果：

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
├── orderer-admin
│   └── msp
│       ├── cacerts
│       │   └── 9-30-250-70-30395-SampleOrgCA.pem
│       ├── keystore
│       │   └── dfe06060490bf62e2bd709433fc747ff28cdbb1e040682c5d47a4e8598db4f2e_sk
│       ├── signcerts
│       │   └── cert.pem
│       └── user
├── peer-admin
│   └── msp
│       ├── cacerts
│       │   └── 9-30-250-70-30395-SampleOrgCA.pem
│       ├── keystore
│       │   └── 24f76217c5c7e641ee29b068712181294e427cbee4dbfce230a1a98b53489cbe_sk
│       ├── signcerts
│       │   └── cert.pem
│       └── user
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

在此 MSP 目录中，打开新用户的 signCert 文件，并使用以下命令将其转换为 Base64 格式：

```
export FLAG=$(if [ "$(uname -s)" == "Linux" ]; then echo "-w 0"; else echo "-b 0"; fi)
cat $HOME/<path-to-peer-admin>/msp/signcerts/cert.pem | base64 $FLAG
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

例如：

```
export FLAG=$(if [ "$(uname -s)" == "Linux" ]; then echo "-w 0"; else echo "-b 0"; fi)
cat $HOME/fabric-ca-client/peer-admin/msp/signcerts/cert.pem | base64 $FLAG
```
{:codeblock}

此命令将显示类似于以下示例的字符串：

```
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNuRENDQWtPZ0F3SUJBZ0lVTXF5VDhUdnlwY3lYR2sxNXRRY3hxa1RpTG9Nd0NnWUlLb1pJemowRUF3SXcKYURFTTlEKaFhTTzRTWjJ2ZHBPL1NQZWtSRUNJQ3hjUmZVSWlkWHFYWGswUGN1OHF2aCtWSkhGeHBLUnQ3dStHZDMzalNSLwotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
```

将此字符串复制到配置文件中 component 部分下的 `"admincerts"` 字段中。

#### 向 TLS CA 注册组件身份
{: #tls-register-component}

您还需要向 TLS CA 注册排序节点或同级。为此，首先需要使用 TLS CA 的管理员进行注册。将 `$FABRIC_CA_CLIENT_HOME` 更改为要存储 TLS CA 管理员证书的目录。

```
cd $HOME/fabric-ca-client
mkdir tlsca-admin
export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca-client/tlsca-admin
```
{:codeblock}

运行以下命令以注册为 TLS CA 的管理员。该命令与用于注册为 [CA 管理员](#enroll-ca-admin)的命令相同，只是您将使用在 [CA 配置](CA_deploy_icp.html#icp-ca-configuration-parms)期间指定的 CA TLS 实例名称。请确保在 CA 私钥中使用相同的 `ca-admin-name` 和 `ca-admin-password`。

```
fabric-ca-client enroll -u https://<enroll_id>:<enroll_password>@<ca_url_with_port> --caname <tls_ca_name> --tls.certfiles <ca_tls_cert_path>
```
{:codeblock}

实际调用可能类似于以下示例：

```
fabric-ca-client enroll -u https://admin:adminpw@9.30.94.174:30167 --caname org1tlsca --tls.certfiles $HOME/fabric-ca-client/catls/tls.pem
```

您注册之后，就有了必需的证书来向 TLS CA 注册组件。运行以下命令注册排序节点或同级：

```
fabric-ca-client register --caname <ca_name> --id.name <name> --id.affiliation org1.department1 --id.type peer --id.secret <password> --tls.certfiles <ca_tls_cert_path>
```
{:codeblock}

此命令类似于用于向 CA 注册组件身份的命令，但使用的是 TLS CA 名称。如果要部署排序节点而不是同级，请将 `--id.type` 设置为 `orderer` 而不是 `peer`。您必须为此身份提供其他用户名和密码，然后将其用于缺省 CA。实际注册可能类似于以下命令：

```
fabric-ca-client register --caname tlsca --id.affiliation org1.department1 --id.name peertls --id.secret peertlspw --id.type peer --tls.certfiles $HOME/fabric-ca-client/catls/tls.pem
```

将 `name` 和 `secret` 的值复制到配置文件的 `"tls"` 部分下的 `"enrollid"` 和 `"enrollsecret"`：

```
"tls": {...
  },
  "enrollid": "peertls",
  "enrollsecret": "peertlspw",
```

### CSR（证书签名请求）主机
{: #csr-hosts}

您需要提供 CSR 主机名以部署排序节点或同级。此主机名包括将部署组件的集群的代理 IP 地址，以及部署 Helm 图表时生成的`服务主机名`。

#### 找到集群代理 IP 地址的值

如果要在部署了 CA 的同一 ICP 集群上部署排序节点或同级，请确保您在将部署该排序节点或同级的 ICP 集群上具有[集群管理员 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/user_management/assign_role.html "集群管理员角色和操作") 角色。然后，输入[配置用于 CA](CA_deploy_icp.html#icp-ca-configuration-parms) 时使用的代理 IP。如果要在其他集群上部署排序节点或同级，那么可以在 ICP 控制台中完成以下步骤来检索集群代理 IP 地址的值：

1. 登录到 ICP 控制台。在左侧导航面板中，单击**平台**，然后单击**节点**以查看在集群中定义的节点。
2. 单击具有角色`代理`的节点，然后从表中复制`主机 IP` 的值。
3. 在以下配置文件的 `"csr"` 部分中，将`主机 IP` 作为 `"hosts"` 的值插入：

  ```
  "csr": {
    "hosts": [""]
  }
  ```
  {:codeblock}

#### 确定服务主机名

部署 Helm 图表时，会生成`服务主机名`。此主机名基于部署 Helm 图表时使用的 `Helm 发布名称`。

- 如果要部署排序节点，那么`服务主机名`是部署期间指定的 `helm_release_name` 在末尾添加字符串 `-orderer` 后的结果。例如，如果集群 IP 代理地址为 `9.42.134.44`，并且指定的 `Helm 发布名称`为 `org1orderer`，那么可以在文件的 `"csr"` 部分中插入以下值：

  ```
  "csr": {
    "hosts": [
       "9.42.134.44",
       "org1orderer-orderer"
     ]
  }
  ```
  {:codeblock}

- 如果要部署同级，那么`服务主机名`是部署期间指定的 `Helm 发布名称`，不带补充字符串。例如，如果集群 IP 代理地址为 9.42.134.44，并且 `Helm 发布名称`为 `org1peer1`，那么可以在 csr 的 "hosts" 中插入以下值：

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

完成上述所有步骤后，更新后的配置文件可能类似于以下示例：

```
{
    "enrollment": {
        "component": {
            "cahost": "9.30.20.70",
            "caport": "32129",
            "caname": "chandra46CA",
            "catls": {
                "cacert": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNGakNDQWIyZ0F3SUJBZ0lVTmlUbkdTandSdU1JVXhpWGcwMGZZWXhPSndJd0NnWUlLb1pJemowRUF3SXcKYURFTE1Ba0dBMVVFQmhNQ1ZWTXhGekFWQmdOVkJBZ1REazV2Y25Sb0lFTmhjbTlzYVc1aE1SUXdFZ1lEVlFRSwpFd3RJZVhCbGNteGxaR2RsY2pFUE1BMEdBMVVFQ3hNR1JtRmljbWxqTVJ0Z3WURWUVFERXhCbVlXSnlhV010ClkyRXRjMlZ5ZG1WeU1CNFhEVEU0TVRFeE5qRTJNVEF3TUZvWERUTXpNVEV4TWpFMk1UQXdNRm93YURFTE1Ba0cKQTFVRUJoTUNWVk14RnpBVkJnTlZCQWdURGs1dmNuUm9JRU5oY205c2FXNWhNUlF3RWdZRFZRUUtFd3RJZVhCbApjbXhsWkdkbGNqRVBNQTBHQTFVRUN4TUdSbUZpY21sak1Sa3dGd1lEVlFRREV4Qm1ZV0p5YVdNdFkyRXRjMlZ5CmRtVnlNRmt3RXdZSEtvWkl6ajBDQVFZSUtvWkl6ajBEQVFjRFFnQUU1dlBucDJUNTdkY2hTOGRLNExsMFJRZEEKd284RmJsMzBPcnBGdWFHUld5TFl4eGcxcVFTemhUY3hTcGtHZjh3a1FzVDVFb01lSWcrRytldjBOU01FUTZORgpNRU13RGdZRFZSMFBBUUgvQkFRREFnRUdNQklHQTFVZEV3RUIvd1FJTUFZQkFmOENBUUV3SFFZRFZSME9CQllFCkZMd2d1N0J3Uk9lQ2hzV2hWQWptMTdmalh1eVBNQW9HQ0NxR1NNNDlCQU1DQTBjQU1FUUNJR0FCRmNSdXhtSkIKY3c4OTJJOXhPS3YxVmYyT0JHZUh5N2pFQzRBRm5najFBaUJqdHFvdjBXMXdxZjhwcGttYkxIQkJoWW1vS3ZqRwo4bDQyeVQ5bWYxWVQrZz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K"
            },
            "enrollid": "peer",
            "enrollsecret": "peerpw",
            "admincerts": [
                "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMzRENDQW9PZ0F3SUJBZ0lVS2Vib0drZEJqenM5bllRbkVQTWp0VG56YTVrd0NnWUlLb1pJemowRUF3SXcKYURFTE1Ba0dBMVVFQmhNQ1ZWTXhGekFWQmdOVkJBZ1REazV2Y25Sb0lFTmhjbTlzYVc1aE1SUXdFZ1lEVlFRSwpFd3RJZVhCbGNteGxaR2RsY2pFUE1BMEdBMVVFQ3hNR1JtRmljbWxqTVJrd0Z3WURWUVFERXhCbVlXSnlhV010ClkyRXRjMlZ5ZG1WeU1CNFhEVEU0TVRFeE9URTNNRE13TUZvWERURTVNVEV4T1RFM01EZ3dNRm93ZmpFTE1BaKQTFVRUJoTUNWVk14RnpBVkJnTlZCQWdURGs1dmNuUm9JRU5oY205c2FXNWhNUlF3RWdZRFZRUUtFd3RJZVhCbApjbXhsWkdkbGNqRXVNQXNHQTFVRUN4TUVkWE5sY2pBTEJnTlZCQXNUQkc5eVp6RXdFZ1lEVlFRTEV3dGtaWEJoCmNuUnRaVzUwTVRFUU1BNEdBMVVFQXhNSFlXUnRhVzR4Y0RCWk1CTUdCeXFHU000OUFnRUdDQ3FHU000OUF3RUgKQTBJQUJLbjUwdEU5TmpZb0RFNDBqalh6RUJ2T2c3Y3RtOElRd0laMnRkS29iNEwwVVhKdSs1Tkt5S2dyUk9vbApWcjBmQmg5cWZWMjl4Nms0T2dmMFNiVklBZWlqZ2ZRd2dmRXdEZ1lEVlIwUEFRSC9CQVFEQWdlQU1Bd0dBMVVkCkV3RUIvd1FDTUFBd0hRWURWUjBPQkJZRUZOYWFkV0VzcGp2Smk1akpiVktiS2M3ZU1wUmhNQjhHQTFVZEl3UVkKTUJhQUZMd2d1N0J3Uk9lQ2hzV2hWQWptMTdmalh1eVBNQ2NHQTFVZEVRUWdNQjZDSEdOb1lXNWtjbUZ6TFcxaQpjQzV5WVd4bGFXZG9MbWxpYlM1amIyMHdhQVlJS2dNRUJRWUhDQUVFWEhzaVlYUjBjbk1pT25zaWFHWXVRV1ptCmFXeHBZWFJwYjI0aU9pSnZjbWN4TG1SbGNHRnlkRzFsYm5ReElpd2lhR1l1Ulc1eWIyeHNiV1Z1ZEVsRUlqb2kKWVdSdGFXNHhjQ0lzSW1obUxsUjVjR1VpT2lKMWMyVnlJbjE5TUFvR0NDcUdTTTQ5QkFNQ0EwY0FNRVFDSURGeApDYzE1bDZUZ1dqYnhSZzlmNjczOGV0K0NZZ1I3VVpGR200VHdoQk5jQWlBNmtUMFFwbDV6WnBUN3BzM0dySXlVCmEydDRHSTQ5ZTdjUm5PMmdrSzl6Z3c9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg=="
            ]
        },
        "tls": {
            "cahost": "9.30.20.70",
            "caport": "32129",
            "caname": "tlsca",
            "catls": {
                "cacert": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNGakNDQWIyZ0F3SUJBZ0lVTmlUbkdTandSdU1JVXhpWGcwMGZZWXhPSndJd0NnWUlLb1pJemowRUF3SXcKYURFTE1Ba0dBMVVFQmhNQ1ZWTXhGekFWQmdOVkJBZ1REazV2Y25Sb0lFTmhjbTlzYVc1aE1SUXdFZ1lEVlFRSwpFd3RJZVhCbGNteGxaR2RsY2pFUE1BMEdBMVVFQ3hNR1JtRmljbWxqTVJrd0Z3WURWUVFERXhCbVlXSnlhV010ClkyRXRjMlZ5ZG1WeU1CNFhEVEU0TVRFeE5qRTJNVEF3TUZvWERUTXpNVEV4TWpFMk1UQXdNRm93YURFTE1Ba0cKQTFVRUJoTUNWVk14RnpBVkJnTlZCQWdURGs1dmNuUm9JRU5oY205c2FXNWhNUlF3RWdZRFZRUUtFd3RJZVhCbApXhsWkdkbGNqRVBNQTBHQTFVRUN4TUdSbUZpY21sak1Sa3dGd1lEVlFRREV4Qm1ZV0p5YVdNdFkyRXRjMlZ5CmRtVnlNRmt3RXdZSEtvWkl6ajBDQVFZSUtvWkl6ajBEQVFjRFFnQUU1dlBucDJUNTdkY2hTOGRLNExsMFJRZEEKd284RmJsMzBPcnBGdWFHUld5TFl4eGcxcVFTemhUY3hTcGtHZjh3a1FzVDVFb01lSWcrRytldjBOU01FUTZORgpNRU13RGdZRFZSMFBBUUgvQkFRREFnRUdNQklHQTFVZEV3RUIvd1FJTUFZQkFmOENBUUV3SFFZRFZSME9CQllFCkZMd2d1N0J3Uk9lQ2hzV2hWQWptMTdmalh1eVBNQW9HQ0NxR1NNNDlCQU1DQTBjQU1FUUNJR0FCRmNSdXhtSkIKY3c4OTJJOXhPS3YxVmYyT0JHZUh5N2pFQzRBRm5najFBaUJqdHFvdjBXMXdxZjhwcGttYkxIQkJoWW1vS3ZqRwo4bDQyeVQ5bWYxWVQrZz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K"
            },
            "enrollid": "peertls",
            "enrollsecret": "peertlspw",
            "csr": {
                "hosts": [
                    "9.30.20.70",
                    "chandra48peer1"
                ]
            }
        }
    }
}
```
{:codeblock}

可以将其他字段保留为空白。保存此文件，在部署[排序节点](/docs/services/blockchain/orderer_deploy_icp.html)或[同级](/docs/services/blockchain/peer_deploy_icp.html)时需要使用此文件。

## 成员资格服务提供者 (MSP)
{: #msp}

{{site.data.keyword.blockchainfull_notm}} Platform 的组件通过成员资格服务提供者 (MSP) 使用身份。MSP 将 CA 颁发的证书与许可权和角色相关联。有关 MSP 的更多信息，请参阅 [Hyperledger Fabric 文档中的 Membership 主题 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/latest/membership/membership.html "Hyperledger Fabric 文档中有关 Membership 的主题")。

MSP 文件夹必须具有定义的结构，供 Fabric 组件使用。Fabric CA 客户机通过创建以下 MSP 文件夹来建立此结构：

- **cacerts：**包含网络的根 CA 的证书。
- **intermediatecerts：**这些是信任链中的任何中间 CA（返回到根 CA）的证书。每个企业套餐组织具有两个中间 CA 以实现故障转移和高可用性。
- **signCerts：**此文件夹包含公共签名证书，也称为 signCert 或注册证书。在从命令行引用 MSP 目录或者使用 SDK 构建用户上下文对象时，此证书连接到网络调用（例如，链代码调用）。如果要通过 SDK 或命令行操作网络，可将此证书上传到 IBP。
- **keystore：**此文件夹包含专用密钥。在命令行中引用 MSP 目录或者使用 SDK 构建用户上下文对象时，此密钥用于对网络调用签名，但它并不像 signCert 那样连接到调用。因此，保证此密钥的安全**至关重要**。如果此密钥泄露，可能会被用于模拟您的身份。

您还可以使用“网络监视器”和 Swagger API 来构建 fabric-ca-client 可引用的 MSP 文件夹。

- **cacerts** 和 **intermediatecerts**：您可以通过向 MSP API 发出 `Get` 请求，使用 [Swagger API](/docs/services/blockchain/swagger_apis.html) 访存这些证书。
- **signCerts** 和 **keystore**：您可以通过单击“认证中心”面板上的**生成证书**按钮，生成这些证书。这样将打开一个列出两个证书的弹出窗口。复制并存储 signCerts 中的**证书**和 keystore 中的**专用密钥**。请将这些证书保存在安全位置，因为这些证书未存储在平台上。

多个 Fabric 组件在其 MSP 文件夹中包含其他信息。例如，如果运行远程同级，那么可能会看到以下文件夹：

- **admincerts：**此文件夹包含此组织或组件的管理员列表。如果从命令行或 SDK 操作远程同级，那么需要将 signCert 上传到此文件夹。如果使用 Fabric CA 客户机，那么在 MSP 中还需要 admincerts 文件夹，该文件夹包含要识别为管理员证书的相应 signCert。
- **tls：**此文件夹存储用于与其他网络组件通信的 TLS 证书。

有关 MSP 结构的更多信息，请参阅 Hyperledger Fabric 文档中的[成员资格 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/latest/membership/membership.html "成员资格") 和[成员资格服务提供者 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/latest/msp.html "成员资格服务提供者")。


## 查看 CA 日志
{: #ca-operate-view-logs}

在命令行中使用 [`kubectl CLI 命令`](#ca-kubectl-configure)或通过 ICP 集群中包含的 [Kibana ![外部链接图标](../images/external_link.svg "外部链接图")](https://www.elastic.co/products/kibana "您使用 Elasticsearch 的窗口")，可以查看组件日志。

- 使用 `kubectl logs` 命令来查看 pod 内的容器日志。如果不确定 pod 名称，请运行以下命令来查看 pod 的列表。

   ```
   kubectl get pods
   ```
   {:codeblock}

   然后，运行以下命令并将 `pod_name` 替换为上面命令输出中 pod 的名称，以检索位于 pod 内的 CA 容器的日志：

   ```
   kubectl logs <pod_name> -c ca
   ```
   {:codeblock}

   有关 `kubectl logs` 命令的更多信息，请参阅 [Kubernetes 文档 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs “Getting Started”)

- 或者，可以使用 [ICP 集群管理控制台](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/troubleshoot/events.html)来访问部署事件和日志，这将在 Kibana 中打开日志。

  **注：**在 Kibana 中查看日志时，可能会收到响应 `No results found`。如果 ICP 将工作程序节点 IP 地址作为其主机名，那么会发生此情况。要解决此问题，请在面板顶部除去以 `node.hostname.keyword` 开头的过滤器，随后相关日志即可显示。

## 故障诊断
{: #ca-operate-troubleshooting}

### **问题：**运行 `enroll` 命令时出错
{: #ca-enroll-error}

运行 Fabric CA 客户机 enroll 命令时，该命令可能会失败并返回以下错误：

```
Error: Failed to read config file at '/Users/chandra/fabric-ca-client/ca-admin/fabric-ca-client-config.yaml': While parsing config: yaml: line 42: mapping values are not allowed in this context
```
{:codeblock}

**解决方案：**

Fabric CA 客户机尝试注册但无法连接到 CA 时，会发生此错误。如果存在以下情况，那么会发生此错误：   

- `enroll` 命令在 `-u` 参数上包含多余的 `https://`。
- CA 名称不正确。
- 您的用户名或密码不正确。

查看在 `enroll` 命令中指定的参数，并确保上述情况都不存在。
