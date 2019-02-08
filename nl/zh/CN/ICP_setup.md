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

# 设置 IBM Cloud Private (ICP)


***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


在 {{site.data.keyword.cloud_notm}} Private (ICP) 上部署 {{site.data.keyword.blockchainfull}} Platform 组件和构建区块链网络之前，需要在您自己的环境中设置 ICP。
{:shortdesc}

## 先决条件
{: #icp-setup-prerequisites}
请完成以下先决条件并准备您的环境以安装 ICP。

### Docker
{{site.data.keyword.cloud_notm}} Private 需要安装 Docker。请遵循[安装 {{site.data.keyword.cloud_notm}} Private ![外部链接图标](/images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/installing/install.html "安装 {{site.data.keyword.cloud_notm}} Private") 中的相关指示信息来安装 Docker。

### ICP 设置
安装 ICP 之前，以下提示有助于准备节点以安装 ICP。其他 ICP 先决条件可以在 [ICP 文档 ![外部链接图标](/images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/installing/prep.html "准备集群以进行安装") 中找到。

#### 更新 `vm.max_map_count` 设置
ICP 使用 Elasticsearch 进行日志记录和测量。为了避免内存不足的异常，Elasticsearch 需要配置 `vm.max_map_count` 系统属性。安装 ICP 之前，请参阅 [Elasticsearch 配置指示信息 ![外部链接图标](/images/external_link.svg "外部链接图标")](https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html "Virtual memory") 以在每个节点上配置此属性。可以使用以下命令来永久设置此属性：

```
sysctl -w vm.max_map_count=262144; sysctl vm.max_map_count
echo "vm.max_map_count=262144” | tee -a /etc/sysctl.conf
```
{:codeblock}

#### 在集群中的每个节点上配置 `/etc/hosts` 文件

- ICP 使用 [Kubernetes ![外部链接图标](images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/tutorials/kubernetes-basics/ "了解 Kubernetes 基础") 来管理容器化应用程序。如果未在每个节点上的 `/etc/hosts` 文件中配置主机名，那么 Kubernetes 域名服务器 (DNS) 会发生故障。在每个节点上的 `/etc/hosts` 文件中[插入集群中每个节点的 IP 地址、主机名和短名称 ![外部链接图标](images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/installing/prep_cluster.html "配置集群")。

- [ICP 不支持 IPv6 ![外部链接图标](images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/getting_started/known_issues.html#ipv6 "IPv6 不受支持")。为了避免 ICP 集群中的 DNS 服务发生问题，请在每个节点上的 `/etc/hosts` 文件中，通过在以下行开头处添加 `#` 号注释掉该行，从而禁用 IPv6 设置：
  ```
  #::1  localhost ip6-localhost ip6-loopback
  ```
  {:codeblock}

### 需要的资源
{: #helm-icp-resources}

确保 ICP 系统满足最低硬件资源需求：

|组件|vCPU|RAM|用于数据存储的磁盘|
|-----------|------|-----|-----------------------|
|CA|1|192 MB|1 GB|
|排序者 (Orderer)|2|512 MB|100 GB，能够扩展|
|同级|2|2 GB|50 GB，能够扩展|
|用于同级的 CouchDB|2|2 GB|50 GB，能够扩展|

 **注：**
 - vCPU 是分配给虚拟机的虚拟核心，或者如果服务器没有针对虚拟机进行分区，那么 vCPU 是物理处理器核心。决定用于 ICP 中部署的虚拟处理器核心 (VPC) 时，需要考虑 vCPU 需求。VPC 是确定 IBM 产品许可成本的计量单位。有关决定 VPC 的场景的更多信息，请参阅 [Virtual processor core (VPC) ![外部链接图标](images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SS8JFY_9.2.0/com.ibm.lmt.doc/Inventory/overview/c_virtual_processor_core_licenses.html)。
 - 这些最低资源级别足以用于测试和试验。对于具有大量交易的环境，请务必分配足够大的存储量；例如，为同级分配 250 GB，为排序服务分配 500 GB。要使用的存储量取决于交易数和网络中所需的签名数。如果同级或排序节点上的存储量即将耗尽，那么必须在更大的文件系统中部署新的同级或排序节点，并使其通过相同通道上的其他组件同步。

#### 存储注意事项
{: #helm-icp-storage-considerations}

* 您需要确定组件将使用的存储器。如果使用缺省设置，同级 Helm 图表将创建一个新的持久卷声明，其名称为 `my-data-pvc`，以用于存储同级数据。如果选择 CouchDB 作为分类帐数据库，那么 Helm 图表将创建另一个持久卷声明，其名称为 `statedb-pvc`，用于存储分类帐数据库。
* 如果不想使用缺省存储设置，请确保在 ICP 安装期间设置*新*的 storageClass，或者 Kubernetes 系统管理员需要在您部署之前创建 storageClass。
* [动态供应 ![外部链接图标](/images/external_link.svg "外部链接图标")]( https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/ "动态卷供应") 仅可用于 ICP 中的 amd64 节点。因此，如果集群包含 s390x 和 amd64 工作程序节点的混合，那么无法使用动态供应。
* 如果未使用动态供应，那么必须创建[持久卷 ![外部链接图标](/images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/storage/persistent-volumes/ "持久卷")，并对其设置标签，标签可用于优化 Kubernetes 持久卷声明 (PVC) 绑定过程。
* 如果您使用 NFS V2/V3 持久卷，那么必须在 NFS 文件系统所在的主机系统上启用 **NFS 状态监视器，以监视 NFS V2/V3 文件系统锁定**模块（也称为 **rpc-statd**）。通过此模块，NFS 文件系统可检查在文件上是否有其他进程持有的互斥锁定。运行以下命令启用此模块：
  ```
  sudo systemctl enable rpc-statd
  ```
  {:codeblock}

  ```
  sudo systemctl start rpc-statd
  ```
  {:codeblock}

## 安装 ICP
{: #icp-setup-install}

完成以下步骤在环境中安装和设置 {{site.data.keyword.cloud_notm}} Private。

1. 安装 [IBM Cloud Private ![外部链接图标](images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/kc_welcome_containers.html) V3.1.0 集群。如果要将 Helm 图表用于开发、测试或实验，那么可以免费安装 [ICP Community Edition V3.1.0 ![外部链接图标](/images/external_link.svg "外部链接图标")]( https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/kc_welcome_containers.html "{{site.data.keyword.cloud_notm}} Private-CE V3.1.0")。

2. 安装 IBM Cloud Private CLI [3.1.0 ![外部链接图标](images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/manage_cluster/install_cli.html) 以安装并运行 CA。

安装 ICP 后，可以继续向 ICP 集群[导入 {{site.data.keyword.blockchainfull_notm}} Platform for ICP Helm 图表](/docs/services/blockchain/howto/helm_install_icp.html)。
