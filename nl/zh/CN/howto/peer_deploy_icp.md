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

# 在 {{site.data.keyword.cloud_notm}} Private 中部署同级
{: #peer-icp}


***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

以下指示信息描述了如何在 {{site.data.keyword.cloud_notm}} Private (ICP) 上部署 {{site.data.keyword.blockchainfull}} Platform 同级。通过这些指示信息，您可以连接到 ICP 上的 {{site.data.keyword.blockchainfull_notm}} Platform。如果要将同级连接到 {{site.data.keyword.cloud_notm}} 上的入门套餐或企业套餐网络，请参阅[部署同级以连接到入门套餐或企业套餐](/docs/services/blockchain/peer_deploy_ibp.html)。
{:shortdesc}

部署同级之前，请查看[注意事项和限制](/docs/services/blockchain/ibp-for-icp-about.html#ibp-icp-considerations)。

## 需要的资源
{: #peer-resources-required}

确保 ICP 系统满足最低硬件资源需求：

|组件|vCPU|RAM|用于数据存储的磁盘|
|-----------|------|-----|-----------------------|
|同级|2|2 GB|50 GB，能够扩展|
|用于同级的 CouchDB|2|2 GB|50 GB，能够扩展|

 **注：**
 - vCPU 是分配给虚拟机的虚拟核心，或者如果服务器没有针对虚拟机进行分区，那么 vCPU 是物理处理器核心。决定用于 ICP 中部署的虚拟处理器核心 (VPC) 时，需要考虑 vCPU 需求。VPC 是确定 IBM 产品许可成本的计量单位。有关决定 VPC 的场景的更多信息，请参阅 [Virtual processor core (VPC) ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SS8JFY_9.2.0/com.ibm.lmt.doc/Inventory/overview/c_virtual_processor_core_licenses.html)。
 - 这些最低资源级别足以用于测试和试验。对于具有大量交易的环境，请务必分配足够大的存储量；例如，为同级分配 250 GB。要使用的存储量取决于交易数和网络中所需的签名数。如果同级或排序节点上的存储量即将耗尽，那么必须在更大的文件系统中部署新的同级或排序节点，并使其通过相同通道上的其他组件同步。

## 存储
{: #storage}

您需要确定同级将使用的存储器。如果使用缺省设置，Helm 图表将创建两个新的 8 Gi 持久卷声明 (PVC)，一个 PVC 的名称为 `my-data-pvc`，用于存储同级数据；另一个 PVC 的名称为 `statedb-pvc`，用于存储状态数据库。

如果不想使用缺省存储设置，请确保在 ICP 安装期间设置*新*的 `storageClass`，否则 Kubernetes 系统管理员需要在您部署之前创建 storageClass。

可以选择在 amd64 或 s390x 平台上部署同级。但是，请注意，[动态供应](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/)仅可用于 ICP 中的 amd64 节点。如果集群包含 s390x 和 amd64 工作程序节点的混合，那么无法使用动态供应。

如果未使用动态供应，那么必须创建[持久卷 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)，并对其设置标签，标签可用于优化 Kubernetes PVC 绑定过程。

## 部署同级的先决条件
{: #prerequisites-peer-icp}

1. 必须先[安装 ICP](/docs/services/blockchain/ICP_setup.html) 并[安装 {{site.data.keyword.blockchainfull_notm}} Platform Helm 图表](helm_install_icp.html)，然后才能在 ICP 上安装同级。

2. 如果使用 Community Edition 并且希望在没有因特网连接的 ICP 集群上运行此 Helm 图表，那么需要在连接因特网的机器上创建归档，然后才能在 ICP 集群上安装归档。有关更多信息，请参阅[向没有因特网连接的集群中添加精选应用程序 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/app_center/add_package_offline.html "向没有因特网连接的集群中添加精选应用程序"){:new_window}。请注意，您可以在 Helm 图表中的 `ibm-blockchain-platform-dev/ibm_cloud_pak` 下找到规范文件 `manifest.yaml`。

3. 必须首先在 ICP 上[部署 CA](/docs/services/blockchain/CA_deploy_icp.html)。您需要使用 CA 来创建[同级配置文件，并将其作为 Kubernetes 私钥存储在 ICP 中](/docs/services/blockchain/#peer-config-file)。

4. 在 ICP 控制台中检索 CA 的集群代理 IP 地址的值。**注：**您需要是[集群管理员 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/user_management/assign_role.html "集群管理员角色和操作") 才能访问代理 IP。登录到 ICP 集群。在左侧导航面板中，单击**平台**，然后单击**节点**以查看在集群中定义的节点。单击具有角色`代理`的节点，然后从表中复制`主机 IP` 的值。**重要信息：**请保存此值，配置 Helm 图表的`代理 IP` 字段时会使用此值。


## 创建同级配置私钥
{: #peer-config-file}

要部署同级，您需要创建配置文件，其中包含有关同级身份和您的认证中心的重要信息。然后，您需要在配置期间使用 [Kubernetes Secret ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/configuration/secret/) 对象将此文件传递到 Helm 图表。通过此文件，同级可从认证中心获取加入区块链网络所需的证书。此文件还包含管理员证书，以允许您操作同级。在配置同级之前，请遵循[使用 CA 部署排序节点或同级](CA_operate.html#deploy-orderer-peer)的指示信息进行操作。

您需要向配置文件提供 CSR 主机名。这包括`服务主机名`，此值与部署期间指定的 `Helm 发布名称`相同。例如，如果将 `Helm 发布名称`指定为 `org1peer1`，请记住在文件的 `"csr"` 部分中插入以下值：
```
"csr": {
  "hosts": [
     "9.42.134.44",
    "org1peer1"
  ]
}
```
{:codeblock}

保存配置文件后，需要先将其编码为 base64 格式，然后再将其作为私钥对象提供给 ICP。Kubernetes 私钥支持您保护和共享信息，而不必将其直接传递到部署。

1. 要将配置文件编码为 Base64 格式，请在终端窗口中运行以下命令：
```
export FLAG=$(if [ "$(uname -s)" == "Linux" ]; then echo "-w 0"; else echo "-b 0"; fi)
cat <config_file> | base64 $FLAG
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

保存生成的输出以供下面的步骤 4 使用。

2. 登录到 ICP 控制台。在左侧导航面板中，单击**配置**，然后单击**私钥**。单击**创建私钥**按钮以打开一个弹出面板，在其中允许您生成新的私钥对象。

3. 在**常规**选项卡上，填写以下字段：
  - **名称：**为私钥提供在集群中唯一的名称。部署同级时将使用此名称。名称必须全部为小写。  
  **注：**部署同级时，部署会自动生成名称为 `<helm_release_name>-secret` 的新私钥。因此，命名私钥时，请确保私钥的名称不同于 `<helm_release_name>-secret`。否则，Helm 图表部署会失败，因为它尝试创建的私钥已存在。
  - **名称空间：**要添加私钥的名称空间。选择要将同级部署到的`名称空间`。
  - **类型：**输入值`不透明`。

4. 单击此窗口的左侧导航窗格上的**数据**选项卡。
  - 在`名称`字段中，指定 `secret.json`，在值字段中，粘贴配置文件的 `base64` 编码的字符串。

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
{: #icp-peer-configuration-parms}

下表列出了 {{site.data.keyword.blockchainfull_notm}} Platform **特定于同级组件**的可配置参数及其缺省值。如果您是试用同级或者是第一次开始使用，请使用数据库和存储参数的缺省值。滚动到同级组件部分以选中`安装同级`复选框，并仅为该组件提供关联的配置信息。**尽管 Helm 图表 UI 指出不需要进一步配置，但您还是需要输入某些参数才能部署同级。**

#### 常规和全局配置参数
{: #peer-global-parameters}

|参数|描述|缺省值|必需|
| --------------|-----------------|-------|------- |
| `Helm 发布名称`|Helm 发布的名称。必须以小写字母开头并以任意字母数字字符结尾，并且只能包含连字符和小写字母数字字符。每次尝试安装组件时，都必须使用唯一的 Helm 发布名称。**重要信息：**此值必须与用于在 [JSON 私钥文件](#peer-config-file)中为“主机”字段生成“服务主机名”的值相匹配。|无|是|
|`目标名称空间`|选择 Kubernetes 名称空间以安装 Helm 图表。|无|是|
|**全局配置**|应用于 Helm 图表中所有组件的参数。|||
|`服务帐户名称`|输入将用于运行 pod 的[服务帐户 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)。|default|否|

#### 同级配置参数
{: #peer-parameters}

|参数|描述|缺省值|必需|
| --------------|-----------------|-------|------- |
|**集群配置**|**集群配置信息**| ||
|`安装同级`|选中以安装同级。|未选中|是，如果要部署同级|
|`同级工作程序节点体系结构`|选择云平台体系结构（AMD64 或 S390x）。|AMD64|是|
|`同级映像存储库`|同级 Helm 图表的位置。此字段会自动填充为安装的路径。|ibmcom/ibp-fabric-peer|是|
|`同级 Docker 映像标记`|与同级映像关联的标记的值。|1.2.1，自动填充为正确的值。|是|
|`同级配置`|可以通过在此字段中粘贴您自己的 `core.yaml` 配置文件来定制同级配置。要查看样本 `core.yaml` 文件，请参阅 [`core.yaml` 样本配置 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://github.com/hyperledger/fabric/blob/release-1.2/sampleconfig/core.yaml)（**仅限高级用户**）。|无|否|
|`同级配置私钥（必需）`|在 ICP 中创建的[同级配置私钥](/docs/services/blockchain/#peer-config-secret)的名称。|无|是|
|`组织 MSP（必需）`|可以创建新的组织 MSPID 值，例如“org1”，也可以指定同级将属于的现有组织 MSP。如果已部署排序节点组织，请确保任何同级 MSPID 都与排序节点 MSPID 不同。另外，请记下此值，因为稍后需要将其用于 `CORE_PEER_LOCALMSPID` 和 `configtx.yaml`。|无|是|
|`同级服务类型`|用于指定是否应该在同级上[公开外部端口 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)。选择 NodePort 以向外部公开端口（建议），选择 ClusterIP 以不公开端口。此发行版中不支持 LoadBalancer 和 ExternalName。|NodePort|是|
|`状态数据库`|用于存储通道分类帐的[状态数据库](/docs/services/blockchain/glossary.html#state-database)。同级需要使用与[区块链网络](/docs/services/blockchain/v10_dashboard.html#network-preferences)相同的数据库。|无|是|
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
{: #icp-helm-cli}

或者，可以使用 `helm` CLI 来安装 Helm 发布。运行 `helm install` 命令之前，请确保[将集群的 Helm 存储库添加到 Helm CLI 环境 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/app_center/add_int_helm_repo_to_cli.html "将内部 Helm 存储库添加到 Helm CLI")。

您可以通过创建 `yaml` 文件并将其传递到以下 `helm install` 命令来设置安装所需的参数。
```
helm install --name <helm_release_name>  <helm_chart> \
--version <helm_chart_version> \
--values <customvalues.yaml> \
--tls
```
{:codeblock}

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
{: #verify-installation-icp}

完成配置参数并单击**安装**按钮后，单击**查看 Helm 发布**按钮可查看部署。如果部署成功，您应该会在“部署”表中看到 `DESIRED`、`CURRENT`、`UP TO DATE` 和 `AVAILABLE` 字段中的值为 1。您可能需要单击“刷新”并等待该表更新。您还可以通过单击 ICP 控制台左上角的**菜单**图标来找到“部署”表。在菜单列表中，单击**工作负载**，然后单击 **Helm 发布**。

如果向下滚动到`注释`部分，可看到在[操作同级](/docs/services/blockchain/peer_operate_icp.html)时将使用的重要信息。

## 查看同级日志
{: #peer-deploy-view-logs}

使用 [kubectl CLI 命令](/docs/services/blockchain/peer_operate_icp.html#peer-kubectl-configure)或通过 [Kibana ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.elastic.co/products/kibana "您使用 Elasticsearch 的窗口") 可以查看同级日志。有关更多信息，请参阅[有关访问日志的指示信息](/docs/services/blockchain/peer_operate_icp.html#peer-icp-view-logs)。

## 下一步做什么

部署同级后，需要完成若干操作步骤，然后才能将交易提交到区块链网络的分布式分类账，并读取该分布式分类账。有关更多信息，请参阅[在使用多云网络的环境中操作同级](/docs/services/blockchain/peer_operate_icp.html)。
