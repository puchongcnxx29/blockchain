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

# 在 {{site.data.keyword.cloud_notm}} Private 上安装 {{site.data.keyword.blockchainfull_notm}} Platform
{: #remote-peer-icp}

{{site.data.keyword.blockchainfull}} Platform for {{site.data.keyword.cloud_notm}} Private 作为 Helm 图表文件提供，可安装在本地 ICP 集群上。安装 Helm 图表之后，您可在 ICP 目录中发现 {{site.data.keyword.blockchainfull_notm}} Platform 显示为应用程序。

在安装 {{site.data.keyword.blockchainfull_notm}} Platform for ICP 之前，请查看[注意事项和限制](/docs/services/blockchain/ibp-for-icp-about.html#ibp-icp-considerations)。有关如何部署 Helm 图表中包含的区块链组件以及有关价格和支持的更多信息，请参阅[关于 {{site.data.keyword.blockchainfull_notm}} Platform for ICP](/docs/services/blockchain/ibp-for-icp-about.html)。

{{site.data.keyword.blockchainfull_notm}} Platform for ICP 提供了两个版本：

- {{site.data.keyword.blockchainfull_notm}} Platform for {{site.data.keyword.cloud_notm}} Private 通过 Passport Advantage (PPA) 提供。您需要拥有所需的许可证以访问 [Passport Advantage Online ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/software/passportadvantage/pao_customer.html "Passport Advantage Online")。购买时，将包含 {{site.data.keyword.blockchainfull_notm}} Platform 技术支持。

- {{site.data.keyword.blockchainfull_notm}} Platform Community Edition 是适用于探索、开发和测试的免费产品。可以通过 [GitHub ![外部链接图标](../images/external_link.svg "外部链接图标")](https://github.com/IBM/charts/tree/master/stable/ibm-blockchain-platform-dev "IBM/charts") 访问此免费版本。请注意，{{site.data.keyword.IBM_notm}} 不提供对 Community Edition 的支持。

## 安装 Helm 图表的先决条件
{: #helm-install-prereqs}

安装 Helm 图表之前，您必须配置 ICP 集群。查看[设置和配置 ICP 集群](/docs/services/blockchain/ICP_setup.html)的指示信息。

##  从防火墙后安装 {{site.data.keyword.blockchainfull_notm}} Platform 
{: #helm-install-prereqs-firewall}

无需因特网连接即可在防火墙后部署 {{site.data.keyword.blockchainfull_notm}} Platform 组件。PPA 包中包含{{site.data.keyword.blockchainfull_notm}} Platform 将使用的所有 Fabric 组件 Docker 映像，您无需在部署期间从 DockerHub 下载这些映像。

但是，Community Edition Helm 图表未包含必需的 Fabric 组件 Docker 映像，因为此版本配置为在部署期间从 DockerHub 下载这些映像。如果没有因特网连接可用，那么部署将失败。因此，您需要完成额外的步骤以在连接因特网的机器上创建归档，然后才能在 ICP 集群上安装这些归档。需要以下映像：
- [Fabric 同级 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hub.docker.com/r/ibmcom/ibp-fabric-peer/ "Farbic 同级")
- [Fabric CA ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hub.docker.com/r/ibmcom/ibp-fabric-ca/ "Fabric CA")
- [Fabric 排序节点 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hub.docker.com/r/ibmcom/ibp-fabric-orderer/ "Fabric 排序节点")
- [Fabric Couch 数据库 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hub.docker.com/r/ibmcom/ibp-fabric-couchdb/ "Fabric couch 数据库")
- [`Init` 映像，用于引导和配置组件，包括 MSP 文件夹 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hub.docker.com/r/ibmcom/ibp-init/)
- [由同级用于运行链代码容器的 Docker-in-Docker (DinD) 映像 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hub.docker.com/r/ibmcom/ibp-dind/)

有关如何使用这些映像的更多信息，请参阅[向没有因特网连接的集群中添加精选应用程序 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/app_center/add_package_offline.html)。请注意，您可以在 Helm 图表中的 `ibm-blockchain-platform-dev/ibm_cloud_pak` 目录下找到规范文件 `manifest.yaml`。

## 将 Helm 图表导入 ICP

1. 从 [Passport Advantage Online ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/software/passportadvantage/pao_customer.html "Passport Advantage Online") 下载 IBM Blockchain Platform for ICP 的 Helm 图表文件或者从 [GitHub ![外部链接图标](../images/external_link.svg "外部链接图标")](https://github.com/IBM/charts/blob/master/repo/stable/ibm-blockchain-platform-dev-1.0.0.tgz "IBM/charts") 下载免费的 Community Edition。此 Helm 图表包中包含三个 Helm 子图表，分别用于 CA、排序节点和同级。

2. 如果尚未登录到 ICP 集群，请进行登录。

  ```
  cloudctl login -a https://<cluster_CA_domain>:8443 --skip-ssl-validation
  ```
  {:codeblock}

4. 确保已配置 [Docker CLI](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/manage_images/configuring_docker_cli.html)。配置 Docker CLI 后，使用以下命令访问集群上的映像注册表：
  ```
  docker login <cluster_CA_domain>:8500
  ```
  {:codeblock}

5. 通过使用以下命令，在 ICP 中查找要上传 Helm 图表的存储库的名称：
  ```
  cloudctl catalog repos
  ```
  {:codeblock}

  成功完成此命令后，您可以看到集群中存储库的列表。选择目标存储库的名称并进行保存。您需要在以下命令中使用该名称。

6. 使用命令行导入 Helm 图表。使用哪个命令导入 Helm 图表取决于 Helm 图表是从 Passport Advantage (PPA) 还是从 GitHub 下载的。

  - **{{site.data.keyword.blockchainfull_notm}} Platform for {{site.data.keyword.cloud_notm}} Private**

    如果从 PPA 下载了 Helm 图表，请遵循以下指示信息。在 ICP CLI 中，从通过 PPA 下载的 Helm 图表包所存储到的目录，运行以下命令，以将 Helm 图表导入 ICP 集群中。

    ```
    cloudctl catalog load-archive --archive <archive-name> --registry <cluster_CA_domain>:8500 --repo <repo-name>
    ```
    {:codeblock}

    替换以下值：
    - `<archive-name>`，替换为所下载的 `.tgz` 文件的名称。
    - `<cluster_CA_domain>:8500`，替换为用于登录 ICP 集群的域。
    - `<repo-name>`，替换为要在其中上传图表的 helm 存储库。运行“cloudctl catalog repos”以列出您的存储库。

    成功完成此命令后，可以看到类似于以下信息的内容：
    ```
    Expanding archive
    OK

    Importing docker images
      Processing image: ibmblockchain/hlfabric-orderer-amd64:1.2.1
      Loading Image
      Tagging Image
      Pushing image as: mycluster.icp:8500/bcaas-usa/ibmblockchain/hlfabric-orderer-amd64:1.2.1
    Pushing image mycluster.icp:8500/bcaas-usa/ibmblockchain/hlfabric-orderer-amd64:1.2.1
      Processing image: ibmblockchain/hlfabric-orderer-s390x:1.2.1
      Loading Image
      Tagging Image
      Pushing image as: mycluster.icp:8500/bcaas-usa/ibmblockchain/hlfabric-orderer-s390x:1.2.1
    Pushing image mycluster.icp:8500/bcaas-usa/ibmblockchain/hlfabric-orderer-s390x:1.2.1
      Processing image: ibmblockchain/hlfabric-ca-amd64:1.2.1
      Loading Image
      Tagging Image
      Pushing image as: mycluster.icp:8500/bcaas-usa/ibmblockchain/hlfabric-ca-amd64:1.2.1
    Pushing image mycluster.icp:8500/bcaas-usa/ibmblockchain/hlfabric-ca-amd64:1.2.1
      Processing image: ibmblockchain/hlfabric-ca-s390x:1.2.1
      Loading Image
      Tagging Image
      Pushing image as: mycluster.icp:8500/bcaas-usa/ibmblockchain/hlfabric-ca-s390x:1.2.1
    Pushing image mycluster.icp:8500/bcaas-usa/ibmblockchain/hlfabric-ca-s390x:1.2.1
      Processing image: ibmblockchain/v1fabric-peer-amd64:1.2.1
      Loading Image
      Tagging Image
      Pushing image as: mycluster.icp:8500/bcaas-usa/ibmblockchain/v1fabric-peer-amd64:1.2.1
    Pushing image mycluster.icp:8500/bcaas-usa/ibmblockchain/v1fabric-peer-amd64:1.2.1
      Processing image: ibmblockchain/v1fabric-peer-s390x:1.2.1
      Loading Image
      Tagging Image
      Pushing image as: mycluster.icp:8500/bcaas-usa/ibmblockchain/v1fabric-peer-s390x:1.2.1
    Pushing image mycluster.icp:8500/bcaas-usa/ibmblockchain/v1fabric-peer-s390x:1.2.1
      Processing image: ibmblockchain/fabric-couchdb-amd64:1.2.1
      Loading Image
      Tagging Image
      Pushing image as: mycluster.icp:8500/bcaas-usa/ibmblockchain/fabric-couchdb-amd64:1.2.1
    Pushing image mycluster.icp:8500/bcaas-usa/ibmblockchain/fabric-couchdb-amd64:1.2.1
      Processing image: ibmblockchain/fabric-couchdb-s390x:1.2.1
      Loading Image
      Tagging Image
      Pushing image as: mycluster.icp:8500/bcaas-usa/ibmblockchain/fabric-couchdb-s390x:1.2.1
    Pushing image mycluster.icp:8500/bcaas-usa/ibmblockchain/fabric-couchdb-s390x:1.2.1
      Processing image: ibmcom/icp-dind-amd64:1.2.1
      Loading Image
      Tagging Image
      Pushing image as: mycluster.icp:8500/bcaas-usa/ibmcom/icp-dind-amd64:1.2.1
    Pushing image mycluster.icp:8500/bcaas-usa/ibmcom/icp-dind-amd64:1.2.1
      Processing image: ibmcom/icp-dind-s390x:1.2.1
      Loading Image
      Tagging Image
      Pushing image as: mycluster.icp:8500/bcaas-usa/ibmcom/icp-dind-s390x:1.2.1
    Pushing image mycluster.icp:8500/bcaas-usa/ibmcom/icp-dind-s390x:1.2.1
      Processing image: ibmcom/icp-busybox-s390x:1.2.1
      Loading Image
      Tagging Image
      Pushing image as: mycluster.icp:8500/bcaas-usa/ibmcom/icp-busybox-s390x:1.2.1
    Pushing image mycluster.icp:8500/bcaas-usa/ibmcom/icp-busybox-s390x:1.2.1
      Processing image: ibmcom/icp-busybox-amd64:1.2.1
      Loading Image
      Tagging Image
      Pushing image as: mycluster.icp:8500/bcaas-usa/ibmcom/icp-busybox-amd64:1.2.1
    Pushing image mycluster.icp:8500/bcaas-usa/ibmcom/icp-busybox-amd64:1.2.1
    OK

    Uploading helm charts
      Processing chart: charts/ibm-blockchain-platform-0.1.5.tgz
      Updating chart values.yaml
      Uploading chart
    Loaded helm chart
    OK

    Synch charts
  Synch started
  OK
  ```

  - **从 GitHub 下载的 Community Edition**
如果从 GitHub 下载了 Helm 图表，请遵循以下指示信息。

    在 ICP CLI 中，从通过 GitHub 下载的 Helm 图表包所存储到的目录，运行以下命令，以将 Helm 图表导入 ICP 集群中。要导入从 GitHub 下载的 Helm 图表，请运行以下命令：
    ```
    cloudctl catalog load-chart --archive <helm_chart_from_github> --repo <repo-name>
    ```
    {:codeblock}

    替换以下值：
    - `<helm_chart_from_github>`，替换为所下载的 .tgz 文件的名称。
    - `<repo-name>`，替换为要在其中上传图表的 helm 存储库。运行“cloudctl catalog repos”以列出您的存储库。

    成功完成此命令后，可以看到类似于以下信息的内容：
    ```
      Loading helm chart
  Loaded helm chart

  

    Synch charts
  Synch started
  OK
  ```

单击 ICP 控制台中的**目录**按钮，然后单击左侧导航面板中的**区块链**，以验证导入是否成功。如果成功，那么 **ibm-blockchain-platform-prod** 或 **ibm-blockchain-platform-dev** 磁贴应该会显示在“ICP 目录”页面上。


## PodSecurityPolicy 需求

将 Helm 图表导入 {{site.data.keyword.cloud_notm}} Private 时，您需要在安装组件之前将 [PodSecurityPolicy ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/concepts/policy/pod-security-policy/ "Pod 安全策略") 绑定到目标名称空间。选择预定义的 PodoSecurityPolicy，或让集群管理员为您创建定制 PodSecurityPolicy：
- 预定义的 PodSecurityPolicy 名称：[`ibm-privileged-psp`](https://ibm.biz/cpkspec-psp)
- 定制的 PodSecurityPolicy 定义：
  ```
  apiVersion: extensions/v1beta1
  kind: PodSecurityPolicy
  metadata:
    name: ibm-blockchain-platform-psp
  spec:
    hostIPC: false
    hostNetwork: false
    hostPID: false
    privileged: true
    allowPrivilegeEscalation: true
    readOnlyRootFilesystem: false
    seLinux:
      rule: RunAsAny
    supplementalGroups:
      rule: RunAsAny
    runAsUser:
      rule: RunAsAny
    fsGroup:
      rule: RunAsAny
    requiredDropCapabilities:
    - ALL
    allowedCapabilities:
    - NET_BIND_SERVICE
    - CHOWN
    - DAC_OVERRIDE
    - SETGID
    - SETUID
    volumes:
    - '*'
  ```
  {:codeblock}
- 用于定制 PodSecurityPolicy 的定制 ClusterRole：
  ```
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRole
  metadata:
    annotations:
    name: ibm-blockchain-platform-clusterrole
  rules:
  - apiGroups:
    - extensions
    resourceNames:
    - ibm-blockchain-platform-psp
    resources:
    - podsecuritypolicies
    verbs:
    - use
  - apiGroups:
    - ""
    resources:
    - secrets
    verbs:
    - create
    - delete
    - get
    - list
    - patch
    - update
    - watch
  ```
  {:codeblock}

- 用于定制 ClusterRole 的定制 ClusterRoleBinding：
  ```
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
   name: ibm-blockchain-platform-clusterrolebinding
  roleRef:
   apiGroup: rbac.authorization.k8s.io
   kind: ClusterRole
   name: ibm-blockchain-platform-clusterrole
  subjects:
  - kind: ServiceAccount
    name: default
    namespace: default
  ```
  {:codeblock}

## 部署个别组件

在安装 Helm 图表之后，在 ICP 目录中单击 **ibm-blockchain-platform-prod** 或 **ibm-blockchain-platform-dev** 磁贴以将其打开。您可以使用配置页面来部署区块链网络的任何个别组件。有关区块链解决方案所需的组件以及部署这些组件时必须采用的顺序的更多详细信息，请参阅 [{{site.data.keyword.blockchainfull_notm}} Platform for ICP 部署指南](/docs/services/blockchain/ibp_for_icp_deployment_guide.html)。

然后部署个别组件：

- 如果要部署排序节点，首先需要为排序节点设置认证中心。CA 会生成将由组织中的其他组件使用的证书。有关更多信息，请参阅[在 ICP 中部署 {{site.data.keyword.blockchainfull_notm}} Platform 认证中心](/docs/services/blockchain/CA_deploy_icp.html)。然后，您可以部署将作为网络公共绑定的排序节点。有关更多信息，请参阅[在 ICP 中部署 {{site.data.keyword.blockchainfull_notm}} Platform 排序节点](/docs/services/blockchain/orderer_deploy_icp.html)

- 如果要部署同级，首先需要为同级设置认证中心。CA 会生成将由同级使用的证书。有关更多信息，请参阅[在 ICP 中部署 {{site.data.keyword.blockchainfull_notm}} Platform 认证中心](/docs/services/blockchain/CA_deploy_icp.html)。然后，当您准备加入网络时，可以部署同级，这些同级将加入通道，对事务背书以及存储数据。有关更多信息，请参阅[在 ICP 中部署 {{site.data.keyword.blockchainfull_notm}} 同级](/docs/services/blockchain/peer_deploy_icp.html)或[部署入门套餐或企业套餐网络的 {{site.data.keyword.blockchainfull_notm}} 同级](/docs/services/blockchain/peer_deploy_ibp.html)，具体取决于同级将加入的区块链网络。
