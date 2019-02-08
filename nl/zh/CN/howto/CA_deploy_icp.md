---

copyright:
  years: 2018
lastupdated: "2018-12-07"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# 在 {{site.data.keyword.cloud_notm}} Private 上部署认证中心
{: #CA-deploy}

***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

导入 {{site.data.keyword.blockchainfull}} Platform on {{site.data.keyword.cloud_notm}} Private (ICP) Helm 图表后，可以部署各个组件。认证中心 (CA) 是组织的信任根，允许您为将部署的其他组件生成凭证。因此，在部署其他组件之前，您需要先部署 CA。多云区块链网络中的每个组织都必须部署自己的 CA。有关 CA 及其在区块链网络中所扮演角色的更多信息，请参阅[认证中心](/docs/services/blockchain/blockchain_component_overview.html#ca)。
{:shortdesc}

部署认证中心之前，请查看[注意事项和限制](/docs/services/blockchain/ibp-for-icp-about.html#ibp-icp-considerations)。

## 需要的资源
{: #ca-resources-required}

确保 ICP 集群满足最低硬件资源需求：

|组件|vCPU|RAM|用于数据存储的磁盘|
|-----------|------|-----|-----------------------|
|CA|1|192 MB|1 GB|

**注：**
- vCPU 是分配给虚拟机的虚拟核心，或者如果服务器没有针对虚拟机进行分区，那么 vCPU 是物理处理器核心。决定用于 ICP 中部署的虚拟处理器核心 (VPC) 时，需要考虑 vCPU 需求。VPC 是确定 IBM 产品许可成本的计量单位。有关决定 VPC 的场景的更多信息，请参阅[虚拟处理器核心 (VPC) ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SS8JFY_9.2.0/com.ibm.lmt.doc/Inventory/overview/c_virtual_processor_core_licenses.html "虚拟处理器核心")。
- 数据存储需求取决于存储的身份和证书的数量。CA 需要的存储量不像同级或排序节点需要的存储量那么多，但这将取决于用例。用户越多，所需的存储量就越多。
- 这些最低资源级别足以用于测试和试验。对于有大量交易的环境，请务必为 CA 分配足够大的存储量。要使用的存储量取决于交易数和网络中所需的签名数。如果 CA 上的存储量即将耗尽，那么必须在更大的文件系统中部署新的 CA，并使其通过相同通道上的其他 CA 同步。

## 存储
{: #storage}

您需要确定 CA 将使用的存储器。如果使用缺省设置，Helm 图表将创建一个新的 2 Gi 持久卷声明 (PVC)，其名称为 `fabric-ca-pvc`，以用于存储 CA。

如果不想使用缺省存储设置，请确保在 ICP 安装期间设置*新*的 `storageClass`，否则 Kubernetes 系统管理员需要在您部署 CA 之前创建 storageClass。

可以选择在 AMD64 或 S390X 平台上部署 CA。但是，请注意，[动态供应](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/)仅可用于 ICP 中的 AMD64 节点。如果集群包含 S390X 和 AMD64 工作程序节点的混合，那么无法使用动态供应。

如果未使用动态供应，那么必须创建[持久卷 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)，并对其设置标签，标签可用于优化 Kubernetes PVC 绑定过程。

## 部署 CA 的先决条件
{: #prerequisites-ca-icp}

1. 必须先[安装 ICP](/docs/services/blockchain/ICP_setup.html) 并[安装 {{site.data.keyword.blockchainfull_notm}} Platform Helm 图表](helm_install_icp.html)，然后才能在 ICP 上安装 CA。

2. 如果使用 Community Edition 并且希望在没有因特网连接的 ICP 集群上运行此 Helm 图表，那么需要在连接因特网的机器上创建归档，然后才能在 ICP 集群上安装归档。有关更多信息，请参阅[向没有因特网连接的集群中添加精选应用程序 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/app_center/add_package_offline.html "向没有因特网连接的集群中添加精选应用程序"){:new_window}。请注意，您可以在 Helm 图表中的 ibm-blockchain-platform-dev/ibm_cloud_pak 下找到规范文件 manifest.yaml。

3. 在 ICP 控制台中检索集群代理 IP 地址的值。**注：**您需要是[集群管理员 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/user_management/assign_role.html "集群管理员角色和操作") 才能访问代理 IP。登录到 ICP 集群。在左侧导航面板中，单击**平台**，然后单击**节点**以查看在集群中定义的节点。单击具有角色`代理`的节点，然后从表中复制`主机 IP` 的值。**重要信息：**请保存此值，配置 Helm 图表的`代理 IP` 字段时会使用此值。

4. 创建 CA 管理员用户名和密码，并将其存储在 ICP 中的私钥对象中。您可以在[下一部分](#admin-secret)中找到创建私钥的步骤。

## 创建 CA 管理员私钥
{: #admin-secret}

CA 首次启动时，会包含您创建用于操作 CA 的管理员身份。部署 CA 之前，需要为此管理员身份创建用户名和密码。请务必记下这些值。稍后在操作 CA 和部署其他网络组件时，会用到这些值。创建 [Kubernetes Secret ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/configuration/secret/ "Secret") 来存储管理员的`用户名`和`密码`以用于 CA 部署。Kubernetes 私钥支持您保护和共享信息，而不必将其直接传递到部署。

1. 创建管理员用户名和密码，并将这些值编码为 Base64 格式。在终端窗口中运行以下命令，并将 `admin` 和 `adminpw` 的值替换为要使用的值。
  ```
  echo -n 'admin' | base64
  echo -n 'adminpw' | base64
  ```
  {:code_block}

  **重要信息：**保存上面创建的 `admin` 和 `adminpw` 的值。另请保存这些字段的编码值，在创建 Kubernetes 私钥对象时需要这些值。

2. 登录到 ICP 控制台。在左侧导航面板中，单击**配置**，然后单击**私钥**。单击**创建私钥**按钮以打开一个弹出面板，在其中允许您生成新的私钥对象。

3. 在**常规**选项卡上，填写以下字段：
  - **名称：**为私钥提供在集群中唯一的名称。配置 CA 时将使用此名称。名称必须全部为小写。
  - **名称空间：**要添加私钥的名称空间。选择要将 CA 部署到的`名称空间`。
  - **类型：**输入值`不透明`。

4. 使**注释**选项卡保留为空白。

5. 在**数据**选项卡中，添加用户名和密码作为键值对。
  1. 在第一个**名称**字段中，输入值 `ca-admin-name`。
  2. 在第一个**值**字段中，输入上面步骤 1 中的结果 `echo -n admin | base64`。
  3. 单击**添加数据**按钮以添加第二个键值对。
  4. 在第二个**名称**字段中，输入值 `ca-admin-password`。
  5. 在第二个**值**字段中，输入上面步骤 1 中的结果 `echo -n adminpw | base64`。  
    **图 1** 显示了 ICP 控制台私钥可能包含的内容，具体取决于您为 `ca-admin-name` 和 `ca-admin-password` 密钥指定的值。

    ![ICP 控制台私钥](../images/CreateCASecret.png "ICP 控制台私钥")  
    *图 1. ICP 控制台私钥*
  6. 单击**创建**以构成新的 Secret 对象。

CA Helm 图表需要使用名称 `ca-admin-name` 和 `ca-admin-password` 作为密钥值，用于将私钥部署到 Helm 发布。

**注：**删除 Helm 发布时，不会从 ICP 集群中除去 CA 管理私钥。您应负责管理 ICP 集群中的私钥。如果您计划未来部署其他 CA，那么可以复用 CA 管理私钥。否则，您应负责将其从 ICP 集群中删除。

## 配置
{: #ca-configuration}

创建 CA 管理私钥后，可以使用以下步骤来配置和安装 CA。一次只能安装一个 CA。

1. 登录到 ICP 控制台，然后单击右上角的**目录**链接。
2. 单击左侧导航面板中的`区块链`，以找到标记为 `ibm-blockchain-platform-prod` 或 `ibm-blockchain-platform-dev`（如果已下载 Community Edition）的磁贴。单击磁贴以将其打开，您可以看到一个自述文件，其中包含有关安装和配置 Helm 图表的信息。
3. 单击面板顶部的**配置**选项卡，或单击右下角的**配置**按钮。
4. 指定[常规配置参数](#ca-global-parameters)的值并接受许可协议。
5. 打开`所有参数`折叠标记，并指定[全局配置参数](#ca-global-parameters)的值。
6. 向下滚动到 **CA 配置**部分。选中`安装 CA` 复选框，并完成 CA 的[配置设置](#ca-parameters)。  
7. 单击**安装**。

### 配置参数
{: #icp-ca-configuration-parms}

下表列出了 {{site.data.keyword.blockchainfull_notm}} Platform **特定于 CA 组件**的可配置参数及其缺省值。  

**尽管 Helm 图表 UI 指出不需要进一步配置，但您还是需要输入某些参数才能部署 CA。**

#### 常规和全局配置参数
{: #ca-global-parameters}

|参数|描述|缺省值|必需|
| --------------|-----------------|-------|------- |
| `Helm 发布名称`|Helm 发布的名称。必须以小写字母开头并以任意字母数字字符结尾，并且只能包含连字符和小写字母数字字符。每次尝试安装组件时，都必须使用唯一的 Helm 发布名称。|无|是|
|`目标名称空间`|选择 Kubernetes 名称空间以安装 Helm 图表。|无|是|
|**全局配置**|应用于 Helm 图表中所有组件的参数。|||
|`服务帐户名称`|输入将用于运行 pod 的[服务帐户 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/ "配置用于 pod 的服务帐户") 的名称。|default|否|

#### CA 配置参数
{: #ca-parameters}

|参数|描述|缺省值|必需|
| --------------|-----------------|-------|------- |
|`安装 CA`|选中以安装 CA。|未选中|是，如果要部署 CA|
|`CA 名称`|指定要用于认证中心的名称。**重要信息：**请记下此值。稍后在配置排序节点或同级时需要此值。|SampleOrgCA|是|
|`CA 工作程序节点体系结构`|选择 ICP 工作程序节点体系结构（AMD64 或 S390X）。|AMD64|是|
|`CA 数据库类型`|用于存储 CA 数据的数据库的类型。仅支持 SQLite。|SQLite|是|
|`CA 数据持久性已启用`|如果选中此项，那么当容器重新启动后，数据将可用。否则，在故障转移或 pod 重新启动时将丢失所有数据。|已选中|否|
|`CA 使用动态供应`|选中此项以对存储卷启用动态供应。|已选中|否|
|`CA 存储类名称`|指定唯一的存储类名称。否则，将使用集群中的缺省存储类。|无|取决于 ICP 集群配置方式。请咨询集群管理员|
|`CA 现有卷声明`|指定现有卷声明的名称，并使其他所有字段保留空白。|无|否|
|`CA 选择器标签`|PVC 的[选择器标签 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/ "标签和选择器")。|无|否|
|`CA 选择器值`|PVC 的[选择器值 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/ "标签和选择器")。|无|否|
|`CA 存储器访问方式`|指定用于 PVC 的存储器[访问方式](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes "访问方式")。|ReadWriteMany|是|
|`CA 卷声明大小`|选择要使用的磁盘的大小。|2 Gi|是|
|`CA 映像存储库`|CA Helm 图表的位置。|ibmcom/ibp-fabric-ca|是|
|`CA Docker 映像标记`|与 CA 映像关联的标记的值。此字段会自动填充为映像版本。请勿对其进行更改。|1.2.1|是|
|`CA 服务类型`|用于指定是否应该在同级上[公开外部端口 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)。选择 NodePort 以向外部公开端口（建议），选择 ClusterIP 以不公开端口。此发行版中不支持 LoadBalancer 和 ExternalName。|NodePort|是|
|`CA 私钥（必需）`|输入为 `ca-admin-name` 和 `ca-admin-password` 创建的 Kubernetes 私钥对象的名称。|无|是|
|`CA CPU 请求`|指定要分配给 CA 的最小 CPU 数。|1|是|
|`CA CPU 限制`|指定要分配给 CA 的最大 CPU 数。|2|是|
|`CA 内存请求`|指定要分配给 CA 的最小内存量。|1 Gi|是|
|`CA 内存限制`|指定要分配给 CA 的最大内存量。|4 Gi|是|
|`CA TLS 实例名称`|指定将用于注册排序节点或同级的 CA TLS 实例的名称。|tlsca|是|
|`CSR 公共名称`|指定生成的 CA 根证书将在联系时提供的公共名称 (CN)。|tlsca-common|是|
|`代理 IP`|输入部署了 CA 的[集群的代理节点 IP ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/installing/install_proxy.html "在 HTTP 代理背后进行 IBM Cloud Private 安装")。|127.0.0.1|是|


### 使用 Helm 命令行安装 Helm 发布
{: #icp-helm-cli}

或者，可以使用 Helm CLI 来安装 Helm 发布。运行 `helm install` 命令之前，请确保[将集群的 Helm 存储库添加到 Helm CLI 环境 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/app_center/add_int_helm_repo_to_cli.html "将内部 Helm 存储库添加到 Helm CLI")。

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
--values ca-s390x-values.yaml \
--tls
```

您可以通过编辑已下载归档文件中包含的 `values.yaml` 来创建新的 `yaml` 文件。`values.yaml` 文件包含所有必需参数及其缺省设置。

## 验证 CA 安装
{: #verifiying-ca-installation}

完成配置参数并单击**安装**按钮后，单击**查看 Helm 发布**按钮可查看部署。如果部署成功，您应该会在“部署”表中看到 `DESIRED`、`CURRENT`、`UP TO DATE` 和 `AVAILABLE` 字段中的值为 1。您可能需要单击“刷新”并等待该表更新。您还可以通过单击 ICP 控制台左上角的**菜单**图标来找到“部署”表。在菜单列表中，单击**工作负载**，然后单击 **Helm 发布**。

如果向下滚动到`注释`部分，可以找到将用于[操作 CA](/docs/services/blockchain/CA_operate.html#ca-operate) 的重要信息。

在 ICP 中安装 {{site.data.keyword.blockchainfull_notm}} Platform CA 后，将使用缺省环境变量设置来创建配置映射。然后，可以更改或添加 CA 服务器的环境变量以配置其行为。有关 CA 服务器配置参数的更多信息，请参阅 [Fabric CA Server 文档 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#fabric-ca-server "Fabric CA Server")。

对配置映射进行配置之后，您需要重新启动 CA 服务器才能使更改生效。要重新启动 CA 服务器，可以删除 Fabric CA 服务器 pod。ICP 将创建反映这些更改的新 pod。

## 查看 CA 日志
{: #ca-deploy-view-logs}

在命令行中使用 [`kubectl CLI 命令`](CA_operate.html#ca-kubectl-configure)或通过 ICP 集群中包含的 [Kibana ![外部链接图标](../images/external_link.svg "外部链接图")](https://www.elastic.co/products/kibana "您使用 Elasticsearch 的窗口")，可以查看组件日志。有关更多信息，请参阅[有关访问日志的指示信息](CA_operate.html#ca-operate-view-logs)。

## 操作 CA
{: #operate-ca}

CA 将是您组织的信任根。您需要使用 CA 来为其他组件生成证书。因此，在部署排序节点或同级之前，必须[设置 CA 并完成若干操作步骤](/docs/services/blockchain/CA_operate.html)。
