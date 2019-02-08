---

copyright:
  years: 2017, 2018
lastupdated: "2018-10-04"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}


# 已知问题


***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


已报告以下问题：
- **尚不支持配置外部 CA**。或者，您可以通过“网络监视器”来生成和上传管理证书。有关更多信息，请参阅[生成客户机端证书](/docs/services/blockchain/v10_application.html#enroll-app)和有关“网络监视器”中[“成员”屏幕的“证书”选项卡](/docs/services/blockchain/v10_dashboard.html#members)的描述。
- 在入门套餐网络的“网络监视器”中，在单击“概述”屏幕上列出的节点的**查看日志**时，将打开 {{site.data.keyword.cloud}} 日志记录 Kibana 界面。**缺省情况下，Kibana 预先配置为显示最近 30 天的活动的日志**。如果最近 30 天内没有任何活动，那么您将看到消息：*找不到结果*。要查看其他日志，您可以单击右上角用户名下面的计时器图标，并设置较大的时间范围，例如，*今年至今*
- 入门套餐网络的日志由 [{{site.data.keyword.cloud_notm}} Log Analysis 服务 ![外部链接图标](images/external_link.svg "外部链接图标")](https://console.bluemix.net/catalog/services/log-analysis) 进行收集。缺省情况下，会通过 Log Analysis 服务的轻量套餐来收集日志。此套餐是免费的，并且**只允许每天搜索日志的前 500 MB**。如果网络日志超过 500 MB，那么无法在 Kibana 中查看新日志。如果网络生成的日志超过 500 MB，您可以升级到 Log Analysis 服务的付费版本。
- 由于入门套餐不是生产环境，所以**应用程序可能无法立即访问网络资源**。
  - 如果发生这种情况，建议首先增大 Fabric SDK 中的缺省超时值。有关设置超时值的更多信息，请参阅[在 Fabric SDK 中设置超时值](/docs/services/blockchain/v10_application.html#set-timeout-in-sdk)。
  - 您还可以在应用程序级别重试请求。
- 因后台网络问题，**链代码容器有时可能会停止**，并且可能需要在用户调用后对其重建并重新启动。如果发生这种情况，您的链代码可能需要几分钟时间才能响应。
- 由于入门套餐网络中的资源限制（即，每个同级 1 个 CPU 和 4 Gi RAM），所以**在链代码实例化期间可能会遇到 `REQUEST_TIMEOUT` 错误**。如果发生这种情况，请重试实例化步骤。如果错误继续存在，可以增大链代码实例化超时。在连接概要文件中，链代码实例化超时设置为 300 秒。
  - 如果要在 SDK 中使用此缺省超时值，请如下所示复制连接概要文件中的 **client** 部分（用于将超时设置为 300 秒），并确保 SDK 对其进行读取。请注意，对于 Node SDK，连接概要文件中的此超时设置会影响所有调用，例如 `invoke`、`queries` 等。
    ```
    "client": {
       "organization": "Org1",
       "connection": {
           "timeout": {
               "peer": {
                   "endorser": "300"
               }
           }
       }
    },
    ```
    {:codeblock}
  - 如果要覆盖 SDK 中链代码实例化命令的超时设置，请将其设置回缺省值或将其更改为 300 秒或更长时间。
    - 如果使用的是 Node SDK，那么可以更改 `sendInstantiateProposal(request, timeout)` 方法的超时设置。有关更多信息，请参阅 [sendInstantiateProposal(request, timeout) ![外部链接图标](images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/Channel.html#sendInstantiateProposal)。
    - 如果使用的是 Java SDK，那么可以更改 `instantiateProposalRequest.setProposalWaitTime(DEPLOYWAITTIME)` 命令的超时设置，该命令位于 `src/test/java/org/hyperledger/fabric/sdkintegration/End2endIT.java` 文件中。

有关 {{site.data.keyword.cloud_notm}} 上 {{site.data.keyword.blockchainfull_notm}} Platform 网络的支持和帮助，请参阅[获取支持](/docs/services/blockchain/ibmblockchain_support.html)。


## 升级企业套餐网络后更新链代码
{: #chaincode-migration}

如果您的企业套餐网络处于 Hyperledger Fabric V1.0 级别，那么 {{site.data.keyword.blockchainfull_notm}} Platform 将安排对您的网络升级，以迁移到 Hyperledger Fabric V1.1。网络升级后，如果原先使用了具有依赖关系的复杂链代码，那么需要更新链代码中的依赖关系。否则，您可能会遇到服务中断。

**注**：
- 如果使用带有单个 `.go` 或 `.js` 文件的简单链代码，那么无需更新链代码。
- 更新后的链代码可能无法在旧网络上运行；因此，在执行安排的网络升级后，需要更新链代码。
- 可以通过在入门套餐网络中安装并实例化链代码来测试链代码。在入门套餐上正常工作的所有链代码在网络升级后，也都会在企业套餐上正常工作。

要更新链代码，请执行以下步骤：
1. 使用任何 Golang vendoring 工具来更新链代码。最简单的方法是使用用于将依赖关系包括在原始文件中的工具。例如，如果链代码使用许多 Fabric 样本所用的 **gosvendor** 工具，那么可以在 vendor 文件夹的上级目录中运行以下命令来更新链代码依赖关系：
    ```
    govendor update all +v
    ```
    {:codeblock}

    然后，可以使用 `go build` 来检查新代码是否已编译以及链代码更新是否工作正常。

2. 网络升级后，可以在“网络监视器”中[更新链代码](/docs/services/blockchain/howto/install_instantiate_chaincode.html#updating-a-chaincode)。
