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

# 在 {{site.data.keyword.cloud_notm}} Private 中部署排序节点
{: #orderer-deploy}

***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

在具有排序节点组件的区块链网络中，排序节点可对客户机进行认证、对事务排序和广播事务。有关排序节点及其在区块链网络中所扮演角色的更多信息，请参阅[区块链组件概述](/docs/services/blockchain/blockchain_component_overview.html)。
{:shortdesc}

部署排序服务之前，请查看[注意事项和限制](/docs/services/blockchain/ibp-for-icp-about.html#ibp-icp-considerations)。

## 需要的资源
{: #ca-resources-required}

确保 ICP 系统满足最低硬件资源需求：

|组件|vCPU|RAM|用于数据存储的磁盘|
|-----------|------|-----|-----------------------|
|排序者 (Orderer)|2|512 MB|100 GB，能够扩展|


 **注：**
 - vCPU 是分配给虚拟机的虚拟核心，或者如果服务器没有针对虚拟机进行分区，那么 vCPU 是物理处理器核心。决定用于 ICP 中部署的虚拟处理器核心 (VPC) 时，需要考虑 vCPU 需求。VPC 是确定 IBM 产品许可成本的计量单位。有关决定 VPC 的场景的更多信息，请参阅 [Virtual processor core (VPC) ![外部链接图标](images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SS8JFY_9.2.0/com.ibm.lmt.doc/Inventory/overview/c_virtual_processor_core_licenses.html)。
 - 这些最低资源级别足以用于测试和试验。对于具有大量交易的环境，请务必分配足够大的存储量；例如，为排序服务分配 500 GB。要使用的存储量取决于交易数和网络中所需的签名数。如果排序节点上的存储量即将耗尽，那么必须在更大的文件系统中部署新的排序节点，并使其通过相同通道上的其他组件同步。

## 存储
{: #storage}

您需要确定排序节点将使用的存储器。如果使用缺省设置，Helm 图表将创建两个新的 8 Gi 持久卷声明 (PVC)，一个 PVC 的名称为 `my-data-pvc`，用于存储排序节点数据；另一个 PVC 的名称为 `statedb-pvc`，用于存储状态数据库。

如果不想使用缺省存储设置，请确保在 ICP 安装期间设置*新*的 `storageClass`，否则 Kubernetes 系统管理员需要在您部署之前创建 storageClass。

可以选择在 amd64 或 s390x 平台上部署排序节点。但是，请注意，[动态供应](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/)仅可用于 ICP 中的 amd64 节点。如果集群包含 s390x 和 amd64 工作程序节点的混合，那么无法使用动态供应。

如果未使用动态供应，那么必须创建[持久卷](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)，并对其设置标签，标签可用于优化 Kubernetes PVC 绑定过程。


## 部署排序节点的先决条件
{: #prerequisites-orderer-icp}

1. 必须先[安装 ICP](/docs/services/blockchain/ICP_setup.html) 并[安装 {{site.data.keyword.blockchainfull_notm}} Platform Helm 图表](helm_install_icp.html)，然后才能在 ICP 上安装排序节点。

2. 如果使用 Community Edition 并且希望在没有因特网连接的 ICP 集群上运行此 Helm 图表，那么需要在连接因特网的机器上创建归档，然后才能在 ICP 集群上安装归档。有关更多信息，请参阅[向没有因特网连接的集群中添加精选应用程序 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/app_center/add_package_offline.html "向没有因特网连接的集群中添加精选应用程序"){:new_window}。请注意，您可以在 Helm 图表中的 ibm-blockchain-platform-dev/ibm_cloud_pak 下找到规范文件 `manifest.yaml`。

3. 在 ICP 控制台中检索 CA 的集群代理 IP 地址的值。**注：**您需要是[集群管理员 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/user_management/assign_role.html "集群管理员角色和操作") 才能访问代理 IP。登录到 ICP 集群。在左侧导航面板中，单击**平台**，然后单击**节点**以查看在集群中定义的节点。单击具有角色`代理`的节点，然后从表中复制`主机 IP` 的值。**重要信息：**请保存此值，配置 Helm 图表的`代理 IP` 字段时会使用此值。

4. 创建[排序节点配置文件，并将其作为 Kubernetes 私钥存储在 ICP 中](#orderer-config-file)。

## 创建排序节点配置文件
{: #orderer-config-file}

部署排序节点之前，您需要创建配置文件，其中包含有关排序节点身份和您的 CA 的重要信息。然后，您需要在配置期间使用 [Kubernetes Secret ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/configuration/secret/) 对象将此文件传递到 Helm 图表。通过此文件，排序节点可从 CA 获取加入区块链网络所需的证书。此文件还包含管理员证书，以允许您以管理员用户身份操作排序节点。在配置排序节点之前，请遵循[使用 CA 部署排序节点或同级](CA_operate.html#deploy-orderer-peer)上的指示信息进行操作。

您需要向配置文件提供 CSR 主机名。这包括`服务主机名`，此值基于部署期间指定的 `Helm 发布名称`。`服务主机名`是指定的 `helm_release_name` 在末尾添加字符串 `-orderer` 后的结果。例如，如果将 `Helm 发布名称`指定为 `orderer1`，那么可以在文件的 `"csr"` 部分中插入以下值：

```
"csr": {
  "hosts": [
    "9.42.134.44",
    "orderer1-orderer"
  ]
}
```

保存配置文件后，需要先将其编码为 Base64 格式，然后再将该文件作为私钥对象提供给 ICP。Kubernetes 私钥支持您保护和共享信息，而不必将其直接传递到部署。

1. 通过在终端窗口中运行以下命令，将配置文件编码为 Base64 格式：

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
  - **名称：**为私钥提供在集群中唯一的名称。部署排序节点时将使用此名称。名称必须全部为小写。  
  **注：**部署排序节点时，部署会自动生成名称为 `<helm_release_name>-orderer-mspsecret` 的新私钥。因此，命名私钥时，请确保私钥的名称不同于 `<helm_release_name>-orderer-mspsecret`。否则，Helm 图表部署会失败，因为它尝试创建的私钥已存在。
  - **名称空间：**要添加私钥的名称空间。选择要将排序节点部署到的`名称空间`。
  - **类型：**输入值`不透明`。

4. 单击此窗口的左侧导航窗格上的**数据**选项卡。在`名称`字段中，指定 `secret.json`，在值字段中，粘贴配置文件的 `base64` 编码的字符串。

5. 单击**创建**以保存私钥对象。

**注：**删除 Helm 发布时，不会从 ICP 集群中除去排序节点配置私钥。如果删除排序节点，那么还需要删除此私钥。

## 配置
{: #orderer-configuration}

创建排序节点配置私钥对象后，可以使用以下步骤在 ICP 中配置和安装排序节点。一次只能安装一个排序节点。

1. 登录到 ICP 控制台，然后单击右上角的**目录**链接。
2. 单击左侧导航面板中的`区块链`，以找到标记为 `ibm-blockchain-platform-prod` 或 `ibm-blockchain-platform-dev`（如果已下载 Community Edition）的磁贴。单击磁贴以将其打开，您可以看到一个自述文件，其中包含有关安装和配置 Helm 图表的信息。
3. 单击面板顶部的**配置**选项卡，或单击右下角的**配置**按钮。
4. 指定[常规配置参数](#orderer-global-parameters)的值并接受许可协议。
5. 打开`所有参数`折叠标记，并指定[全局配置参数](#orderer-global-parameters)的值。
6. 向下滚动到**排序节点配置**部分。选中`安装排序节点`复选框，并完成排序节点的[配置设置](#orderer-parameters)。
7. 单击**安装**。


**注：**将排序节点部署到 s390x 时，初始化容器可能会返回以下错误：

`panic: Error while trying to open DB: no locks available`

这是因为排序节点使用的是平面文件数据库，而 NFS 文件系统需要额外的包，以使排序服务能够查询文件来检查这些文件上是否存在互斥锁定以及查询文件来创建互斥锁定。要解决此问题，必须启用 `rpc-statd` 包。

`sudo systemctl enable rpc-statd`
`sudo systemctl start rpc-statd`

必须在提供 NFS 文件系统的系统上完成此操作。

### 配置参数
{: #icp-orderer-configuration-parms}

下表列出了 {{site.data.keyword.blockchainfull_notm}} Platform **特定于排序节点组件**的可配置参数及其缺省值。**尽管 Helm 图表 UI 指出不需要进一步配置，但您还是需要输入某些参数才能部署排序节点。**

#### 常规和全局配置参数
{: #orderer-global-parameters}

|参数|描述|缺省值|必需|
| --------------|-----------------|-------|------- |
| `Helm 发布名称`|Helm 发布的名称。必须以小写字母开头并以任意字母数字字符结尾，并且只能包含连字符和小写字母数字字符。每次尝试安装组件时，都必须使用唯一的 Helm 发布名称。**重要信息：**此值必须与用于在 [JSON 私钥文件](#orderer-config-file)中为 "hosts" 字段生成“服务主机名”的值相匹配。|无|是|
|`目标名称空间`|选择 Kubernetes 名称空间以安装 Helm 图表。|无|是|
|**全局配置**|应用于 Helm 图表中所有组件的参数。|||
|`服务帐户名称`|输入将用于运行 pod 的[服务帐户 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)。|default|否|

#### 排序节点配置参数
{: #orderer-parameters}

|参数|描述|缺省值|必需|
| --------------|-----------------|-------|------- |
|`安装排序节点`|选中以安装排序节点。|未选中|是，如果要部署排序节点|
|`排序节点工作程序节点体系结构`|选择 ICP 工作程序节点体系结构（AMD64 或 S390X）。|基于主节点自动检测的体系结构|是|
|`排序节点配置`|可以通过在此字段中粘贴您自己的 `orderer.yaml` 配置文件来定制排序节点配置。要查看样本 `orderer.yaml` 文件，请参阅 [`orderer.yaml` 样本配置 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://github.com/hyperledger/fabric/blob/release-1.2/sampleconfig/orderer.yaml)（**仅限高级用户**）。|无|否|
|`组织 MSP 私钥（必需）`|指定包含组织 MSP 证书和密钥的私钥对象的名称。|无|是|
|`排序节点数据持久性已启用`|容器重新启动后，数据将可用。如果未选中此项，那么在故障转移或 pod 重新启动时将丢失所有数据。|已选中|否|
|`排序节点使用动态供应`|选中此项以对存储卷启用动态供应。|已选中|否|
|`排序节点映像存储库`|排序节点 Helm 图表的位置。此字段会自动填充为安装的路径。如果您使用的是 Community Edition，但无法访问因特网，请将此字段更改为 Fabric 排序节点映像的下载位置。|ibmcom/ibp-fabric-orderer|否|
|`排序节点 Docker 映像标记`|Docker 映像的记录。此字段会自动填充为映像版本。请勿对其进行更改。|1.2.1|是|
|`排序节点共识类型`|排序服务的共识类型。|SOLO|是|
|`排序节点组织名称`|指定要用于排序节点组织的名称。如果还计划部署同级，请确保使用的名称与您将为同级提供的名称不同。例如，为排序节点组织提供 `ordererOrg` 之类的名称|无|是|
|`排序节点组织 MSP 标识`|指定要用于排序节点组织的 MSP 标识的名称。此名称应该与您为排序节点组织提供的名称相同，并且会被部署过程设置为环境变量。请记下此值，稍后需要引用此值。|无|是|
|`排序节点存储类名称`|指定排序节点的存储类名称。|无|取决于 ICP 集群配置方式。请咨询集群管理员|
|`排序节点现有卷声明`|指定现有卷声明的名称，并使其他所有字段保留空白。|无|否|
|`排序节点选择器标签`|PVC 的[选择器标签 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)。|无|否|
|`排序节点选择器值`|PVC 的[选择器值 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)。|无|否|
|`排序节点存储器访问方式`|指定用于 PVC 的存储器[访问方式 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes)。|ReadWriteMany|是|
|`排序节点卷声明大小`|选择要使用的磁盘大小，必须至少为 2 Gi。|8 Gi|是|
|排序节点服务类型|用于指定是否应该在同级上[公开外部端口 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)。选择 NodePort 以向外部公开端口（建议），选择 ClusterIP 以不公开端口。此发行版中不支持 LoadBalancer 和 ExternalName。|NodePort|是|
|`排序节点 CPU 请求`|指定要分配给排序节点的最小 CPU 数。|1|是|
|`排序节点 CPU 限制`|指定要分配给排序节点的最大 CPU 数。|2|是|
|`排序节点内存请求`|指定要分配给排序节点的最小内存量。|1 Gi|是|
|`排序节点内存限制`|指定要分配给排序节点的最大内存量。|2 Gi|是|

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
--values orderer-s390x-values.yaml \
--tls
```

您可以通过编辑已下载归档文件中包含的 `values.yaml` 来创建新的 `yaml` 文件，values.yaml 文件包含所有必需参数及其缺省设置。

## 验证排序节点安装

完成配置参数并单击**安装**按钮后，单击**查看 Helm 发布**按钮可查看部署。如果部署成功，您应该会在“部署”表中看到 `DESIRED`、`CURRENT`、`UP TO DATE` 和 `AVAILABLE` 字段中的值为 1。您可能需要单击“刷新”并等待该表更新。您还可以通过单击 ICP 控制台左上角的**菜单**图标来找到“部署”表。在菜单列表中，单击**工作负载**，然后单击 **Helm 发布**。

## 查看排序节点日志
{: #orderer-view-logs}

在命令行中使用 [`kubectl CLI 命令`](orderer_operate.html#orderer-kubectl-configure)或通过 ICP 集群中包含的 [Kibana ![外部链接图标](../images/external_link.svg "外部链接图")](https://www.elastic.co/products/kibana "您使用 Elasticsearch 的窗口")，可以查看组件日志。有关更多信息，请参阅[有关访问日志的指示信息](orderer_operate.html#orderer-view-logs)。
