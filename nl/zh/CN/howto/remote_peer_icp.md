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

# 在 {{site.data.keyword.cloud_notm}} Private 中部署远程同级
{: #remote-peer-icp}


***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


以下指示信息描述了如何安装 {{site.data.keyword.blockchainfull}} Platform Remote Peer on {{site.data.keyword.cloud_notm}} Private (ICP)，并将其连接到 {{site.data.keyword.blockchainfull_notm}} Platform 上的网络。
{:shortdesc}

{{site.data.keyword.cloud_notm}} Private 是用于开发和管理内部部署容器化应用程序的应用程序平台。{{site.data.keyword.blockchainfull_notm}} Platform Remote Peer on {{site.data.keyword.cloud_notm}} Private 是一个 {{site.data.keyword.blockchainfull_notm}} 产品，与 ICP 捆绑在一起，供客户在自己的本地环境中部署同级。安装远程同级的 Helm 图表之后，您可在 ICP 目录中找到显示为应用程序的该项。ICP 的远程同级利用 ICP 的存储器、安全性、日志记录和支持服务进行同级管理。有关 ICP 的更多信息，请参阅 [{{site.data.keyword.cloud_notm}} Private 文档 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0.3/kc_welcome_containers.html "{{site.data.keyword.cloud_notm}} Private 文档")。

在 ICP 上安装远程同级之前，请查看[远程同级注意事项](/docs/services/blockchain/remote_peer.html#remote-peer-limitations)。

## 先决条件
{: #prerequisites-icp}

在 ICP 上安装远程同级之前，必须在 {{site.data.keyword.cloud_notm}} Private、本地系统和 {{site.data.keyword.blockchainfull_notm}} Platform 上完成以下步骤。

1. 本地系统

  需要安装 {{site.data.keyword.cloud_notm}} Private CLI [2.1.0.3 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.3/manage_cluster/install_cli.html "安装 {{site.data.keyword.cloud_notm}} Private CLI")。  
<!---
    - Fabric-ca binaries.
      1. Download the [fabric-ca binaries](https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric-ca/hyperledger-fabric-ca/) to your local machine and extract them.
      2. Move the executable `fabric-ca` client from the directory where it was downloaded into a new `fabric-ca-remote` directory.
         ```
         mkdir $HOME/fabric-ca-remote
         cp {downloads}/bin/fabric-ca-client $HOME/fabic-ca-remote/
         ```
         {:codeblock}
--->
2. {{site.data.keyword.cloud_notm}} Private

  需要 [{{site.data.keyword.cloud_notm}} Private ![外部链接图标](../images/external_link.svg "外部链接图标")]( https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.3/kc_welcome_containers.html "{{site.data.keyword.cloud_notm}} Private V2.1.0.3 文档") 集群 V2.1.0.3。如果要使用远程同级来进行开发、测试或实验，那么可以免费安装 [{{site.data.keyword.cloud_notm}} Private Community Edition ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0.3/installing/install_containers_CE.html "安装 {{site.data.keyword.cloud_notm}} Private-CE")。

  **注**：
    - 您需要确定同级将使用的存储器。如果使用缺省设置，Helm 图表将创建一个新的 8 Gi 持久卷声明，其名称为 `my-data-pvc`，以用于存储同级数据。如果选择 CouchDB 作为分类帐数据库，那么 Helm 图表将创建另一个 8 Gi 持久卷声明，其名称为 `statedb-pvc`，用于存储分类帐数据库。
    - 如果不想使用缺省存储设置，请确保在 ICP 安装期间设置*新*的 storageClass，或者 Kubernetes 系统管理员需要在您部署之前创建 storageClass。
    - [动态供应 ![外部链接图标](../images/external_link.svg "外部链接图标")]( https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/ "动态卷供应") 仅可用于 ICP 中的 amd64 节点。因此，如果集群包含 s390x 和 amd64 工作程序节点的混合，那么无法使用动态供应。
    - 如果未使用动态供应，那么必须创建[持久卷 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/storage/persistent-volumes/ "持久卷")，并使用标签对其进行设置，标签可用于优化 Kubernetes 持久卷声明 (PVC) 绑定过程。

  除了 ICP 硬件需求外，远程同级还至少需要以下各项：
    - 2 个 vCPU
    - 2 GB RAM
    - 用于链代码的 4 GB 空间
    - 用于分类帐的 10 GB 空间，并能够随着分类帐扩大而增大

3. {{site.data.keyword.blockchainfull_notm}} Platform  

  要使用远程同级，您必须拥有一个组织，该组织是在 {{site.data.keyword.blockchainfull_notm}} Platform 上托管的区块链网络的成员。您需要使用 {{site.data.keyword.cloud_notm}} 上的“网络监视器”来访问网络的网络凭证和 API 端点。如果您不是任何区块链网络的成员，那么需要创建或加入网络。有关更多信息，请参阅[创建网络](/docs/services/blockchain/get_start.html#creating-a-network)或[加入网络](/docs/services/blockchain/get_start.html#joining-a-network)。

## 将 Helm 图表导入 ICP

{{site.data.keyword.blockchainfull_notm}} Platform Remote Peer 软件包在 [Github ![外部链接图标](../images/external_link.svg "外部链接图标")]( https://github.com/IBM/charts/tree/master/stable/ibm-blockchain-platform-remote-peer "IBM Blockchain Remote Peer") 中提供。Helm 图表可使用命令行导入 ICP 控制台。在本地计算机中，完成以下步骤以导入 Helm 图表：

1. 从 [Github ![外部链接图标](../images/external_link.svg "外部链接图标")](https://github.com/IBM/charts/tree/master/repo/stable/ibm-blockchain-platform-remote-peer-0.9.0.tgz "IBM Blockchain Remote Peer Helm 图表") 下载 `.tgz` Helm 图表文件。<!--[Passport Advantage Online ![External link icon](../images/external_link.svg "External link icon")](https://www-01.ibm.com/software/passportadvantage "Passport Advantage Online")-->

2. 如果您尚未登录，请通过 ICP CLI 登录到 ICP 集群，然后登录到 Docker 专用映像注册表。  
  ```
  bx pr login -a https://<cluster_CA_domain>:8443 --skip-ssl-validation
  docker login <cluster_CA_domain>:8500
  ```
  {:codeblock}

  在该代码中，`<cluster_CA_domain>` 是认证中心 (CA) 域。如果未指定 CA 域，那么缺省值为 `mycluster.icp`。

3. 在 ICP CLI 中，从存储所下载 Helm 图表的目录中运行以下命令，以将 Helm 图表导入 ICP 集群。将 `<helm_chart_from_github>` 替换为已从 GitHub 下载的 `.tgz` 文件的名称。

  ```
  bx pr load-helm-chart --archive <helm_chart_from_github> --clustername <cluster_CA_domain>
  ```
  {:codeblock}

  成功完成此命令后，可以看到类似于以下信息的内容：

  ```  
  Loading helm chart
  Loaded helm chart

  Synch charts
  Synch started
  OK
  ```  

  现在，“{{site.data.keyword.blockchainfull_notm}} Platform Remote Peer Helm 图表”磁贴应该会显示在“ICP 目录”页面上。单击 ICP 控制台中的**目录**按钮可验证导入是否成功。

### 部署远程同级

以下步骤描述了在 ICP 中部署远程同级时所需的过程。在 ICP 中配置远程同级之前，请检索有关 {{site.data.keyword.blockchainfull_notm}} Platform 的一些端点信息，并为远程同级创建用户身份。您需要在后续步骤中使用此信息。

## 步骤 1：检索远程同级配置信息
{: #network-endpoints}

您需要在配置期间向远程同级提供网络的 API 端点。这些端点允许远程同级查找并连接到 {{site.data.keyword.blockchainfull_notm}} Platform 上的网络。在“网络监视器”的**概述**屏幕上，单击**远程同级配置**按钮。

![远程同级配置](../images/myresources_starter.png "远程同级配置")
*图 1. 远程同级配置*

这将打开一个弹出窗口，其中显示以下字段的值。保存这些值，以在配置远程同级时使用。

  - **网络标识**
  - **组织 MSP**
  - **认证中心 (CA) 名称**
  - **认证中心 (CA) URL**
  - **认证中心 (CA) TLS 证书**

您可以分别复制和粘贴字段，或者通过单击**下载**链接将其另存为 JSON 文件。**注**：如果以 JSON 格式下载这些信息，那么在将 TLS 证书提供给远程同级之前，需要先将其转换为 PEM 格式。可以使用 CLI 通过发出 `echo -e "<CERT>" > catls.pem` 命令，将证书转换为 PEM 格式。

## 步骤 2：注册远程同级
{: #register-peer}

需要向 {{site.data.keyword.blockchainfull_notm}} Platform 上的组织添加新的同级身份后，远程同级才能加入网络。完成以下步骤以注册远程同级。

1. 登录到 {{site.data.keyword.blockchainfull_notm}} Platform 并访问“网络监视器”。在“认证中心”屏幕上，可以查看已向组织注册的所有身份，例如管理员或客户机应用程序。
  ![CA 屏幕](../images/CA_screen_starter.png "CA 屏幕")
  *图 2. CA 屏幕*

2. 单击屏幕顶部的**添加用户**按钮。这将打开一个弹出屏幕，并允许您在填写以下字段后注册远程同级。保存“标识”和“私钥”的值，在配置同级时需要使用这些值。
  - **标识：**同级的使用名称，在配置同级时称为`注册标识`。**保存此值**，以供未来部署远程同级时使用。
  - **私钥：**同级的密码，在配置同级时称为`注册私钥`。**保存此值**，以供未来部署远程同级时使用。
  - **类型：**对于此字段，选择`同级`。
  - **亲缘关系：**这是远程同级将属于的组织（例如，`org1`）下的亲缘关系。从下拉列表中选择现有亲缘关系，或输入新的亲缘关系。
  - **最大注册数：**可以使用此字段来限制可以通过此身份注册或生成证书的次数。如果未指定，那么值会缺省为无限次数注册。

  填写完字段后，单击**提交**以注册远程同级。随后，注册的同级会以组织中的身份在表中列出。

## 步骤 3：配置和安装远程同级

在“网络监视器”中注册同级身份后，可以使用以下 ICP 控制台指示信息在 ICP 中配置和安装远程同级。

<!-- Issue #228 or the [helm CLI](#icp-helm-cli). -->

1. 登录到 ICP 控制台，然后单击右上角的**目录**链接。
2. 找到 {{site.data.keyword.blockchainfull_notm}} Platform Remote Peer 的 Helm 图表，然后单击以将其打开。您可以看到包含安装和配置 Helm 图表相关信息的自述文件。
3. 单击**配置**。在“配置”页面上，填写 Helm 图表的配置设置，并确认您已阅读并同意许可协议。
4. 单击**安装**。

### 配置参数
{: #icp-configuration-parms}

您需要提供远程同级连接到 {{site.data.keyword.blockchainfull_notm}} Platform 上的网络所需的信息。下表列出了 {{site.data.keyword.blockchainfull_notm}} Platform Remote Peer 图表的可配置参数及其缺省值。


|参数|描述|缺省值|必需|
| --------------|-----------------|-------|------- |
|**配置**|**为远程同级配置发布名称和目标名称空间**|  ||
|`发布名称`|Helm 发布的名称。|无|是|
|`目标名称空间`|Kubernetes 名称空间。|default|是|
| | | | |
|**集群配置**|**集群配置信息**| ||
|`工作程序节点体系结构`|选择云平台体系结构（amd64 或 S390x）。|基于主节点自动检测的体系结构|是|
|`映像`|Helm 图表的路径。|自动填充到安装路径，请勿更改此值|是|
| | | | |
|**区块链网络**|**远程同级所需的网络配置信息**| | |
|`网络标识`|在 {{site.data.keyword.blockchainfull_notm}} Platform“网络监视器”中找到的网络标识的值。单击“概述”屏幕上的`远程同级配置`按钮，然后将该信息复制并粘贴到此处。|无|是|
|`组织 MSP`|在 {{site.data.keyword.blockchainfull_notm}} Platform“网络监视器”中找到的组织 MSP 标识的值。单击“概述”屏幕上的`远程同级配置`按钮，然后将该信息复制并粘贴到此处。|无|是|
|`认证中心 (CA) 名称`|在 {{site.data.keyword.blockchainfull_notm}} Platform“网络监视器”中找到的 CA 名称的值。单击“概述”屏幕上的`远程同级配置`按钮，然后将该信息复制并粘贴到此处。|无|是|
|`认证中心 (CA) URL`|在 {{site.data.keyword.blockchainfull_notm}} Platform“网络监视器”中找到的 CA URL 的值。单击“概述”屏幕上的`远程同级配置`按钮，然后将该信息复制并粘贴到此处。|无|是|
|`认证中心 (CA) TLS 证书`|{{site.data.keyword.blockchainfull_notm}} Platform“网络监视器”中的 CA TLS 证书字符串。单击“概述”屏幕上的`远程同级配置`按钮，然后将该信息复制并粘贴到此处。|无|是|
| | | | |
|**远程同级身份**|**用于注册远程同级的注册标识和私钥**| | |
|`同级注册标识`|这是您在 {{site.data.keyword.blockchainfull_notm}} Platform“网络监视器”的“认证中心”屏幕上输入的注册标识。|无|是|
|`同级注册私钥`|这是您在 {{site.data.keyword.blockchainfull_notm}} Platform“网络监视器”的“认证中心”屏幕上输入的注册私钥。|无|是|
| | | | |
|**远程同级数据库**|**分类帐数据库类型**| | |
|`分类帐数据库`|用于存储通道分类帐的[状态数据库](/docs/services/blockchain/glossary.html#state-database)。远程同级需要使用与[区块链网络](/docs/services/blockchain/v10_dashboard.html#network-preferences)相同的数据库。|无|是|
|`CouchDB 映像`|仅当选择了 CouchDB 作为分类帐数据库时才适用。|自动填充到安装路径，请勿更改此值。|是|
|**数据持久性**|启用在集群重新启动或发生故障后持久存储数据的功能。有关更多信息，请参阅 [Kubernetes 中的存储器 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/storage/ "卷")。| | |
|`数据持久性已启用`|容器重新启动后，状态数据将可用。*如果未选中此项，那么在故障转移或 pod 重新启动时将丢失所有数据。*|已选中|否|
|`使用动态供应`|使用 Kubernetes 动态存储器。|已选中|否|
| | | | |
|**持久卷配置**|**将由同级使用的持久卷声明**|  |  |
|`持久卷声明`|输入将创建的新持久性卷声明 (PVC) 的名称。|my-data-pvc|否|
|`存储类名称`|选择存储类名称。|如果创建新的 PVC，那么为空白；否则，请指定与现有 PVC 关联的存储类。|否|
|`现有卷声明`|指定现有卷声明的名称，并使其他所有字段保留空白。|新的声明名称|否|
|`存储器访问方式`|指定用于 PVC 的存储器[访问方式 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes "访问方式")。|ReadOnlyMany|否|
|`选择器标签`|PVC 的[选择器标签 ![外部链接图标](../images/external_link.svg "外部链接图标") ](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/ "标签和选择器")。|default|否|
|`选择器值`|PVC 的[选择器值 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/ "标签和选择器")。|default|否|
|`卷声明大小`|卷声明的大小必须大于 2 Gi。|8 Gi|是|
| | | | |
|**状态数据库数据持久性**|**容器重新启动或发生故障后持久存储数据。有关更多信息，请参阅 [Kubernetes 中的存储器 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/storage/ "卷")** | | |
|`数据持久性已启用`|容器重新启动后，分类帐数据将可用。*如果未选中此项，那么在故障转移或 pod 重新启动时将丢失所有数据。*|已选中|否|
|`使用动态供应`|使用 Kubernetes 动态存储器。|已选中|否|
| | | | |
|**状态数据库持久卷配置**|**要用于状态数据库的持久卷声明。有关更多信息，请参阅 [Kubernetes 中的存储器 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/storage/ "卷")。** |||
|`持久卷声明`|为将创建的新 CouchDB 持久卷声明输入名称或使用缺省值。|statedb-pvc|否|
|`状态数据库存储类名称`|选择存储类名称。|无|否|
|`状态数据库现有卷声明`|要使用现有卷声明，请输入 PVC 名称，并使其他所有字段保留空白。|无|否|
|`状态数据库存储器访问方式`|指定用于 PVC 的存储器[访问方式 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes "访问方式")。|ReadOnlyMany|否|
|`选择器标签`|PVC 的[选择器标签 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/ "标签和选择器")。|default|否|
|`选择器值`|PVC 的[选择器值 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/ "标签和选择器")。|default|否|
|`状态数据库卷声明大小`|持久卷声明的大小。|8 Gi|否|
| | | | |
|**同级资源**|**同级容器的最小及最大 CPU 和内存**| | |
|`同级 CPU 请求`|分配给同级的最小 CPU 数。|2|是|
|`同级 CPU 限制`|分配给同级的最大 CPU 数。|2|是|
|`同级内存请求`|分配给同级的最小内存量。|4 Gi|是|
|`同级内存限制`|分配给同级的最大内存量。|4 Gi|是|
| | | | |
|**CouchDB 资源**|**分配给 CouchDB 容器的最小及最大 CPU 和内存。仅当选择了 CouchDB 作为状态数据库时，此部分才适用**| | | |
|`CouchDB CPU 请求`|分配给 CouchDB 的最小 CPU 数。|2|是|
|`CouchDB CPU 限制`|分配给 CouchDB 的最大 CPU 数。|2|是|
|`CouchDB 内存请求`|分配给 CouchDB 的最小内存量。|4 Gi|是|
|`CouchDB 内存限制`|分配给 CouchDB 的最大内存量。|4 Gi|是|

选择了 CouchDB 作为远程同级数据库时，将在 pod 中创建两个容器，一个容器用于同级，另一个容器用于 CouchDB。同级容器包含同级 PVC 的单个卷安装，用于在文件系统上存储区块和事务。CouchDB 容器会安装包含分类帐数据的同级状态数据库 PVC。

选择了 LevelDB 作为远程同级数据库时，将在 pod 中创建一个容器，用于运行同级和 LevelDB 进程。此容器有两个卷安装，一个卷安装用于同级 PVC，另一个卷安装用于包含分类帐数据的同级状态数据库 PVC。

|远程同级数据库选择|容器 #1 的内容|容器 #2 的内容|
| --------------|-----------------|---------------|
|CouchDB|安装了同级 PVC 的远程同级|安装了状态数据库 PVC 的 CouchDB|
|LevelDB|安装了同级 PVC 和状态数据库 PVC 的远程同级和 LevelDB|不适用|

<!-- Issue #228

### Using the helm command line to install the helm release
{: #icp-helm-cli}

Alternatively, you can use the helm CLI to install the helm release. Before running the helm install command, complete these
[steps ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0.3/app_center/add_int_helm_repo_to_cli.html "Adding the internal Helm repository to Helm CLI") to add your cluster's helm repository to the helm CLI environment.

To install a release with the default configuration use the `helm install` CLI command, replacing the following values in `< >` based on parameter information in the table below.

```
helm install --name <release_name>  <helm_chart> \
--version <helm_chart_version> \
--set env.networkid=<network_id> \
--set env.key=<API_key> \
--set env.secret=<API_secret> \
--set peeridentity.peerUser=<peer_user> \
--set peeridentity.peerPassword=<peer_password> \
--set dataPVC.size=<data_volume_claim_size> \
--set couchdbPVC.size=<couchDB_volume_claim_size> \
--tls
```

where:
- `<release name>` represents the name you want to give your helm release.
- `<helm_chart>`  represents the name of the Helm Chart imported into the catalog.
- `<helm_chart_version>` represents the version of the Helm Chart imported into the catalog.


For example:
```
helm install --name jnchart2 mycluster/ibm-blockchain-platform-remote-peer-prod-1.0.0 \
--version 1.0.2 \
--set env.networkid=6f037b0186064edd9f0032ksl2dcd3c6ee \
--set env.key=PeerOrg1 \
--set env.secret=8s1MOzVd9YxmdmRvm_owlYXX0Gr5XGx30ol2o0vNo18Zvba5jM3xCIOuQ-yEGBn \
--set peeridentity.peerUser=peer621 \
--set peeridentity.peerPassword=peer621 \
--set dataPVC.size=8Gi \
--set couchdbPVC.size=8Gi \
--tls
```

Optionally, a `.yaml` file containing the parameters can be passed to the `helm install` command
using the following format:

```
helm install --name <release_name>  <helm_chart> \
--version <helm_chart_version> \
--values <customvalues.yaml> \
--tls
```

where `--values <customvalues.yaml>` could also be expressed as `-f <customvalues.yaml>`.

The `values.yaml` file includes all of the parameter names and is included in the archive downloaded from
Docker Hub.

-->

<!-- Commenting out information per issue #228

**Note:** The following error can occur when running the `helm install` command if the
helm CLI version on your local machine is ahead of helm cli version on your server.

```
Error: incompatible versions client[v2.9.1] server[v2.7.3+icp]
```

See the [troubleshooting topic](remote_peer_operate_icp.html#icp-troubleshooting) for instructions on resolving this error.

-->


<!--
### Generating your own peer TLS certificates
{: #generate-peer-tls-certs}

You can leave the `Peer TLS CA certificate`, `Peer TLS certificate`, and `Peer TLS key` fields empty or provide your own certificates.

- If you leave all these three fields empty, your remote peer will generate a set of default values for these fields when the remote peer starts.

  **Note:** If you specify values for one or two fields of these three fields, the remote peer will generate and use the default values for all three fields.

- If you don't want to use the default values, you can generate your own certificates and key and specify them in the three fields.

  **Note:** When you specify your own certificates in the three fields, ensure that the value is in one line. You can connect the content from each line with `\r\n`. For example, your certificate value might look like the following example.

  ```
  -----BEGIN CERTIFICATE-----
  MIIEkDCCAnigAwIBAgIBCDANBgkqhkiG9w0BAQsFADAiMQswCQYDVQQGEwJVUzET
  MBEGA1UEAwwKdGVzdGNhLmNvbTAeFw0xODA4MDMwMzM2MDFaFw0yMTA0MjkwMzM2
  MDFaMCMxCzAJBgNVBAYTAlVTMRQwEgYDVQQDDAt0ZXN0b3JnLmNvbTCCASIwDQYJ
  4kyRt4UAWNvEswnLcaYCtS4ZYa8=
  -----END CERTIFICATE-----
  ```

  You need to use `\r\n` to connect all lines as shown below.

  ```
  ----BEGIN CERTIFICATE-----\r\nMIIEkDCCAnigAwIBAgIBCDANBgkqhkiG9w0BAQsFADAiMQswCQYDVQQGEwJVUzET\r\nMBEGA1UEAwwKdGVzdGNhLmNvbTAeFw0xODA4MDMwMzM2MDFaFw0yMTA0MjkwMzM2\r\nMDFaMCMxCzAJBgNVBAYTAlVTMRQwEgYDVQQDDAt0ZXN0b3JnLmNvbTCCASIwDQYJ\r\n4kyRt4UAWNvEswnLcaYCtS4ZYa8=\r\n-----END CERTIFICATE-----
  ```

Before you generate certificates, you need to create a folder that contains the `openssl-ca.cnf` and `openssl-server.cnf` files, and an empty `index.txt` file.

If you haven't, install [openssl](https://github.com/openssl/openssl) on your system.

1. Run the following command to generate **Peer TLS CA certificates**.
  ```
  openssl req -x509 -newkey rsa:4096 -sha256 -nodes -out ./cacert.pem -keyout ./cakey.pem -outform PEM  -config openssl-ca.cnf -subj "/C=US/CN=testca.com"
  ```

  This command generates the `cacert.pem` and `cakey.pem` files. The `cacert.pem` file is your Peer TLS CA certificate.

2. Run the following command to generate **Peer TLS key**.
  ```
  openssl req -newkey rsa:2048 -sha256 -nodes -out ./servercert.csr -outform PEM -keyout ./peer-key.pem -config openssl-server.cnf  -subj "/C=US/CN=testorg.com"
  ```

  This command generates `peer-key.pem` and `servercert.csr` files. The `peer-key.pem` file is your Peer TLS key.

3. Run the following command to generate **Peer TLS certificate**.
  ```
  openssl ca -batch  -config ./openssl-ca.cnf -policy signing_policy -extensions signing_req -keyfile ./cakey.pem -cert ./cacert.pem -out ./peer-cert.pem -infiles ./servercert.csr
  ```

  此命令会生成 `peer-cert.pem` 文件，这是同级 TLS 证书。

--->

## 步骤 4：验证远程同级安装
{: #verify-installation-icp}

安装远程同级后，可以查看同级日志以检查同级安装是否成功。

1. 单击**安装**后，单击弹出窗口中的**查看 Helm 发布**按钮。向下滚动到 **pod** 部分。在 pod 状态显示为**正在运行**时，可以在**部署**部分下找到您的远程同级实例。

2. 在 ICP 控制台中，单击左上角的**菜单**图标。从菜单列表中，单击**工作负载** > **部署**。在部署表中，单击您创建的远程同级实例，这将打开部署概述屏幕。单击**日志**选项卡以查看远程同级日志。可以在搜索字段中输入字符串 `Started`。

  - 如果远程同级已成功启动，那么可以看到类似于以下示例的日志：
    ```
    [36m2018-06-25 14:22:36.929 UTC [inproccontroller] func2 -> DEBU 196[0m chaincode-support started for qscc-1.1.0
    [36m2018-06-25 14:22:36.929 UTC [inproccontroller] func1 -> DEBU 197[0m chaincode started for qscc-1.1.0
    2018-06-25 14:22:36.942 UTC [nodeCmd] serve -> INFO 1cc[0m Started peer with ID=[name:"fabric-peer" ], network ID=[fa74d88bbd9f46a48a6e4c9986e84228], address=[10.1.156.81:7051]
    ```

  - 如果未在屏幕中看到任何日志，说明远程同级未成功启动。要查看其他日志以弄清楚问题，请单击左上角的**菜单**图标，然后单击**工作负载** > **Helm 发布**。单击 Helm 发布以将其打开。单击关联 **pod** 旁边的**查看日志**链接，以在 Kibana 界面中查看其他图表日志。
3. 要验证远程同级是否可以连接到 {{site.data.keyword.blockchainfull}} Platform 网络，可以从远程同级容器内运行同级 CLI 命令。运行 `peer channel fetch` CLI 命令以从通道访存起源区块：

   1. 如果尚未连接到 ICP 集群，请遵循以下[指示信息](/docs/services/blockchain/remote_peer_operate_icp.html#remote-peer-kubectl-configure)以连接到该集群，并在同级容器内使用 CLI。

   2. 您需要先将组织添加到网络中的通道，然后才能访存该起源区块。

     - 可以为远程同级启动新的通道。作为通道启动者，您可以在[通道创建](/docs/services/blockchain/create_channel.html#creating-a-channel)期间自动包含您的组织。请注意，您需要在 {{site.data.keyword.blockchainfull_notm}} Platform 上至少具有一个同级后，才能在“网络监视器”中创建通道。  

     - 区块链网络的其他成员也可以使用[通道更新](/docs/services/blockchain/create_channel.html#updating-a-channel)将您的组织添加到现有通道。{{site.data.keyword.blockchainfull_notm}} Platform 上同级的通道成员可以使用“网络监视器”将您的组织添加到通道，即使您未在该平台上托管任何同级。
      在安装期间，远程同级将上传其签名证书，因此您只需要将该证书同步到通道即可。在“网络监视器”的“通道”屏幕中，找到组织加入的通道，然后从**操作**标头下的下拉列表中选择**同步证书**。此操作会同步通道上所有同级的证书。

   3. 从“网络监视器”的“概述”屏幕上提供的 `creds.json` 文件中检索配置信息。单击**连接概要文件**，然后单击**下载**。

     - 通过搜索`排序者 > URL` 下的**排序者**来查找排序者 URL。记下以网络名称开头的 URL。该 URL 类似于以下示例：
       ```
       ash-zbc07b.4.secure.blockchain.ibm.com:21239
       ```

     - 通过搜索**组织**来查找组织的名称。这应该是您用于注册远程同级的组织。您可以找到组织的名称及其关联的 `mspid`。请记下 `mspid` 的值。

    4. 运行以下命令以在同级容器中设置环境变量。

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
         - 将 `<PEER_ADDR>` 替换为 `localhost:7051`。

       例如：

       ```
       export ORDERER_1=ash-zbc07b.4.secure.blockchain.ibm.com:21239
       export CHANNEL=defaultchannel
       export CC_NAME=mycc
       export ORGID=PeerOrg1
       export PEERADDR=localhost:7051
       ```
       {:codeblock}

   5. 运行以下同级 CLI 命令以访存通道的起源区块。

     ```
     CORE_PEER_TLS_ROOTCERT_FILE=/mnt/certs/tls/cacert.pem CORE_PEER_TLS_ENABLED=true CORE_PEER_ADDRESS=${PEERADDR} CORE_PEER_LOCALMSPID=${ORGID} CORE_PEER_MSPCONFIGPATH=/mnt/crypto/peer/peer/msp/ GOPATH=/ peer channel fetch 0 -o ${ORDERER_1} -c ${CHANNEL} --cafile /mnt/certs/tls/cacert.pem --tls
     ```
     {:codeblock}

     **注：**运行其中任一 CLI 命令时，可能会遇到以下警告，您可以安全地忽略此警告。

     ```
     [msp] getPemMaterialFromDir -> WARN 001 Failed reading file
     /mnt/crypto/peer/peer/msp/intermediatecerts/<intermediate cert name>.pem: no pem content for file  /mnt/crypto/peer/peer/msp/intermediatecerts/<intermediate cert name>.pem
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

## 后续步骤
{: #whats-next-icp}

在 ICP 中设置远程同级后，可以完成若干操作步骤，然后才能将事务提交到区块链网络的分布式总账，并读取该分布式总账。有关更多信息，请参阅[操作远程同级](/docs/services/blockchain/remote_peer_operate_icp.html)。

## 高可用性
{: #high-availability}

作为高可用性最佳实践，强烈建议您每个组织至少部署两个同级。

完成以下步骤以在 ICP 上部署高可用性远程同级：

1. 查看如何[配置 ICP 集群以实现高可用性 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.3/installing/high_availability.html "高可用性 {{site.data.keyword.cloud_notm}} Private 集群")。
2. 在集群上多次安装同级，以将多个同级部署到不同的工作程序节点。Helm 图表包含基于网络标识和 OrgID 的反亲缘关系策略。ICP 集群会检测同级是否从同一组织连接到同一网络，并尝试在不同的工作程序节点上安排这些同级。  
  **注：**虽然策略“首选”将多个同级置于不同的工作程序节点上，但并不会强制这样做。例如，如果您只有一个工作程序节点，那么所有同级都会部署在这同一个节点上。

如果部署了至少两个同级，那么还需要配置[应用程序以实现高可用性](/docs/services/blockchain/v10_application.html#ha-app)。

## 安全性注意事项
{: #icp-security}

远程同级部署在 {{site.data.keyword.blockchainfull}} Platform 外部；因此，管理远程同级的安全性是您的责任。这包括企业套餐网络提供的重要安全性领域，例如密钥管理和数据加密。考虑远程同级的安全性时，请查看以下主题。

### 身份和访问权控制
{: #icp-security-node}

在 {{site.data.keyword.cloud_notm}} 外部，您还需要为与远程同级交互的用户设置身份管理、认证和访问控制。可以使用 [ICP 用户管理 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.3/user_management/access_overview.html "安全性") 来配置这些功能。

### 数据安全性
{: #icp-security-data}

{{site.data.keyword.blockchainfull}} Platform 企业套餐使用的是基于[对称密钥加密 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSB23S_1.1.0.14/gtps7/s7symm.html "对称加密") 的全盘加密来保护网络使用的所有数据。您必须在自己的环境中执行类似的步骤来保护远程同级数据。

### 数据存储位置
{: #icp-data-residency}

数据存储位置要求所有区块链分类帐数据的处理和存储始终保持在一个国家或地区的境内执行。
有关如何实现数据存储位置的更多信息，请参阅[数据存储位置](/docs/services/blockchain/remote_peer.html#data-residency)。

### 密钥管理
{: #icp-security-key-management}

密钥管理是远程同级安全性中至关重要的一个方面。如果专用密钥泄漏或丢失，那么恶意参与者可能有能力访问远程同级的数据和功能。{{site.data.keyword.blockchainfull_notm}} Platform 企业套餐使用[硬件安全模块](/docs/services/blockchain/glossary.html#hsm) (HSM) 来存储网络的专用密钥。HSM 是一种物理设备，用于阻止其他参与方访问您的专用密钥。
在 ICP 上部署远程同级时，管理专用密钥是您的责任。虽然 {{site.data.keyword.blockchainfull_notm}} Platform 会生成专用密钥，但这些密钥不会存储在 Platform 上。因此，必须确保将密钥存储在安全的位置，避免密钥泄露。您可以在同级 MSP 的密钥库文件夹（即同级容器的 `/mnt/crypto/peer/peer/msp/keystore/` 目录）中找到远程同级的专用密钥。有关远程同级内的证书的更多信息，请参阅[在 {{site.data.keyword.blockchainfull_notm}} Platform 上管理证书](/docs/services/blockchain/certificates.html)教程中的[成员资格服务提供者](/docs/services/blockchain/certificates.html#msp)部分。

您可以使用“密钥托管”来恢复丢失的专用密钥。这需要在丢失任何密钥之前执行。如果无法恢复专用密钥，那么需要通过向认证中心注册新的身份来获取新的专用密钥。您还应该从加入的任何通道中除去并替换 signCert。

<!---
在 IBP 中创建专用密钥时，将生成两组独立的密钥资料，以监管两个不同的实体。然后，会将这两组密钥资料组合在一起以创建专用密钥。
--->

### TLS
{: #icp-security-tls}

[传输层安全性 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_7.1.0/com.ibm.mq.doc/sy10660_.htm "SSL 或 TLS 握手概述") (TLS) 嵌入在 Hyperledger Fabric 的信任模型中。{{site.data.keyword.blockchainfull_notm}} Platform 上的所有组件都使用 TLS 进行认证并相互通信。因此，{{site.data.keyword.blockchainfull_notm}} Platform 上的网络组件需要能够完成与远程同级的 TLS 握手。这意味着多项操作，其中一个操作是您需要在客户机应用程序到同级之间的 Web 防火墙中启用通过（例如，使用白名单来实现）。

可以使用需要双向（服务器-客户机）而不是单向（仅服务器）认证的相互 TLS 来保护应用程序网络和企业套餐网络之间的通信。您可以使用 {{site.data.keyword.blockchainfull_notm}} Platform 上的“网络监视器”为同级[启用相互 TLS](../v10_dashboard.html#mutual-tls)。要在远程同级上启用相互 TLS，请遵循 Hyperledger Fabric 文档中[为同级节点启用相互 TLS ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.1/enable_tls.html "使用传输层安全性保护通信") 的指示信息。强烈建议您为应用程序启用相互 TLS。

### 成员资格服务提供者配置
{: #icp-security-MSP}

{{site.data.keyword.blockchainfull_notm}} Platform 的组件通过成员资格服务提供者 (MSP) 来使用身份。MSP 将 CA 颁发的证书与网络和通道角色相关联。有关 MSP 如何用于远程同级的更多信息，请参阅[成员资格服务提供者 (MSP)](/docs/services/blockchain/certificates.html#msp)。

### 应用程序安全性
{: #icp-security-appl}

由于所有链代码调用都已签名，因此 Fabric 会管理应用程序安全性。此外，Fabric 还包含基于 ACL 的应用程序级别检查。

## 许可和定价
{: #license-pricing-icp}

{{site.data.keyword.blockchainfull_notm}} Platform Remote Peer on {{site.data.keyword.cloud_notm}} Private 是一个免费 Beta 版本，适用于评估和实验，并作为 ICP 应用程序进行部署。您可以访问 [Github 软件包 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://github.com/IBM/charts/tree/master/stable/ibm-blockchain-platform-remote-peer "{{site.data.keyword.blockchainfull_notm}} Platform Remote Peer")<!--[IBM Passport Advantage Online ![External link icon](../images/external_link.svg "External link icon")](https://www-01.ibm.com/software/passportadvantage) to purchase the license and-->并下载 [Helm 图表![外部链接图标](../images/external_link.svg "外部链接图标")](https://github.com/IBM/charts/tree/master/repo/stable/ibm-blockchain-platform-remote-peer-0.9.0.tgz "{{site.data.keyword.blockchainfull_notm}} Platform Remote Peer Helm 图表")。

<!--Purchasing the remote peer on IBM Passport Advantage includes the license, a Helm Chart installation package, a Quick Start Guide, and a default level support for {{site.data.keyword.blockchainfull_notm}} Platform Remote Peer.-->
