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

# 在 {{site.data.keyword.cloud_notm}} Private 上操作排序节点
{: #orderer-operate}

***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

在 ICP 中安装 {{site.data.keyword.blockchainfull}} Platform 排序节点后，将创建包含环境变量缺省设置的配置映射。然后，可以更改或添加排序节点的环境变量以配置其行为。

一般而言，排序节点管理员负责引导和维护排序节点，但在只有一个排序节点存在的 SOLO 部署中，这是不必要的。在 SOLO 部署中，排序节点管理员负责将新组织添加到排序节点系统通道。

## 先决条件

您需要从 ICP 集群完成一些必备步骤才能操作排序节点。

**先决条件：**
- [配置 CLI](#orderer-kubectl-configure)
- [检索排序节点端点信息](#orderer-endpoint)
- [下载排序节点 TLS 证书](#orderer-tls)

然后，您可以使用本主题中的指示信息来操作排序节点。请注意，您将通过命令行操作排序节点，这需要获取 `peer` CLI 二进制文件。`peer` CLI 二进制文件与其他二进制文件（例如，`configtxlator`）一起下载。因此，本主题中的许多命令都以“peer”这个词开头。实际上这并不是使用同级，而只是二进制文件的名称。

### 配置 CLI
{: #orderer-kubectl-configure}

您需要使用 **kubectl** 命令行工具来连接到在 ICP 中运行的排序节点容器。

1. 登录到 ICP 集群 UI。浏览至**命令行工具**选项卡，然后单击 **Cloud Private CLI**。您将看到以下可以下载的工具。

   * 安装 IBM Cloud Private CLI 和插件
   * 安装 Kubectl CLI
   * 安装 Helm
   * 安装 Istio CLI

  要操作排序节点，您需要使用前**三个**工具，其中 Helm 是可选的。单击每个工具，然后针对您使用的机器类型，运行相应的 `curl` 命令。接下来，对每个工具发出 `chmod` 和 `sudo mv` 命令。`chmod` 命令将更改相关 CLI 的许可权以使其可执行，`sudo mv` 命令将移动文件并对其重命名。

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

  如果该命令成功，将返回类似于以下示例的响应，其中 `orderer-6cf85f6687-hcxpl` 是排序节点容器的名称。

  ```
  NAME                                 READY      STATUS             RESTARTS   AGE
  orderer-6cf85f6687-hcxpl          2/2        Running            0          6h
  ```

  您现在已准备就绪，可以使用 **kubectl** 工具来获取排序节点端点信息。

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

### 检索排序节点端点信息
{: #orderer-endpoint}

您需要将排序节点端点设定为目标，才能对排序节点系统通道进行更新。您需要是[集群管理员 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/user_management/assign_role.html "集群管理员角色和操作") 才能完成以下步骤：

1. 登录到 ICP 控制台，然后单击左上角的**菜单**图标。
2. 单击**工作负载** > **Helm 发布**。
3. 找到您的 Helm 发布的名称，然后打开 Helm 发布详细信息面板。
4. 向下滚动到面板底部的**注释**部分。**注释**部分包含一组命令，可帮助您操作排序节点部署。
5. 如果尚未配置 [kubectl CLI](#orderer-kubectl-configure)，请遵循相关指示信息进行配置。您需要使用该 CLI 与排序节点容器进行交互。
6. 在 CLI 中，运行注释中的第一个命令，此命令位于 **1. 通过运行以下命令获取应用程序 URL：**之后。此命令将显示排序节点 URL 和端口，类似于以下示例。保存此 URL，并在未来的命令中使用此 URL 来设置代理地址和外部节点端口。

  ```
  http://9.30.94.174:30159
  ```

在此示例中，代理 IP 地址为 `9.30.94.174`，与 7050 相对应的外部节点端口是 `30159`。  

**注：**如果将排序节点部署在防火墙后，那么需要打开传递，以支持平台上的网络完成此任务。

### 下载排序节点 TLS 证书
{: #orderer-tls}

您需要下载排序节点 TLS 证书，并将其传递到命令，才能通过远程客户机与排序节点进行通信。您需要是[集群管理员 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/user_management/assign_role.html "集群管理员角色和操作") 才能完成以下步骤：

1. 登录到 ICP 控制台，然后单击左上角的**菜单**图标。
2. 单击**工作负载** > **Helm 发布**。
3. 找到您的 Helm 发布的名称，然后打开 Helm 发布详细信息面板。
4. 向下滚动到面板底部的**注释**部分。**注释**部分包含一组命令，可帮助您操作排序节点部署。
5. 如果尚未配置 [kubectl CLI](#ca-kubectl-configure)，请遵循相关指示信息进行配置。您需要使用该 CLI 与排序节点容器进行交互。
6. 在 CLI 中，运行注释中的第三个命令，此命令位于 **3. 获取排序节点 TLS 根证书文件**之后。此命令会将 TLS 证书作为 `cert.pem` 文件保存在本地计算机上。

  **注：**在运行这些命令之前，可以先除去 `app=orderer`，如以下命令所示：

  ```
  export POD_NAME=$(kubectl get pods --namespace blockchain-dev -l "release=pa-orderer3" -o jsonpath="{.items[0].metadata.name}")
  ```
  {:codeblock}

7. 将生成的证书移至可在未来的命令中引用该证书的位置，然后将其重命名为 `orderertls.pem`。

  ```
  mkdir $HOME/fabric-ca-client/orderer-tls
  cp cert.pem $HOME/fabric-ca-client/orderer-tls/orderertls.pem
  ```
  {:codeblock}

### 管理本地系统上的证书
{: #manage-certs}

切换到在其中生成排序节点管理员的 MSP 文件夹的目录。根据您执行本文档中示例步骤的方式，或者根据要部署的组件数，您可以在 `$HOME/fabric-ca-client/orderer-admin/msp` 或 `$HOME/fabric-ca-client/peer-admin/msp` 中找到 MSP 文件夹。

需要在本地计算机上对证书执行一些管理工作，才能操作排序节点。您还需要确保可以从排序节点访问 TLS 证书。有关要使用的证书的更多信息，请参阅[在 {{site.data.keyword.cloud_notm}} Private 上操作认证中心](/docs/services/blockchain/CA_operate.html)中的[成员资格服务提供者](/docs/services/blockchain/CA_operate.html#msp)。

1. 将排序节点管理员的 signCert 移至名为 `admincerts` 的新文件夹：

  ```
  cd $HOME/fabric-ca-client/orderer-admin/msp
  mkdir admincerts
  cp signcerts/cert.pem admincerts/cert.pem
  ```
  {:codeblock}

2. 确保您已[下载排序节点 TLS 证书](#orderer-tls)，并且可以通过命令行引用该证书。如果遵循的是本文档中的示例命令，那么可以在 `$HOME/fabric-ca-client/orderer-tls/orderertls.pem` 文件中找到此 TLS 证书。

您可以运行 tree 命令来验证是否已完成这些步骤。浏览至存储证书的目录。tree 命令应该生成类似于以下结构的结果：
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
│       ├── admincerts
│       │   └── cert.pem
│       ├── cacerts
│       │   └── 9-30-250-70-30395-SampleOrgCA.pem
│       ├── keystore
│       │   └── dfe06060490bf62e2bd709433fc747ff28cdbb1e040682c5d47a4e8598db4f2e_sk
│       ├── signcerts
│       │   └── cert.pem
│       └── user
├── orderer-tls
│   └── orderertls.pem
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

## 向排序节点系统通道添加组织
{: #add-organizations-to-consortium}

**注：**如果您计划部署同级并将其连接到入门套餐或企业套餐网络，那么可以跳过此步骤。

使用 {{site.data.keyword.blockchainfull_notm}} Platform for ICP Helm 图表部署排序节点时，会以系统通道唯一管理员身份为排序节点组织自动创建起源区块。这意味着，只有排序节点组织管理员有能力向联盟添加组织。在将新组织添加到排序节点系统通道后，不应更改管理此能力的策略（称为排序节点系统通道的 `mod_policy`）。

通过将组织添加到排序节点系统通道，可使这些组织成为“联盟”的成员，联盟是缺省情况下可以创建通道的成员的列表。成为联盟的成员后，还可以使用证书和系统通道中列出的信息轻松将成员添加到通道中。

更新排序节点系统通道通过“配置更新事务”来实现，其中组织的相关 MSP 信息会添加到排序节点系统通道的通道配置中。

将组织添加到排序节点系统通道本质上是与更新任何通道的配置以添加组织相同的流程。但是，需要进行一些更改，因为要更新的通道不是应用程序通道，并且相关管理员是排序节点管理员，而不是同级组织。

请注意，您无需首先加入系统通道就可以向通道添加组织。有关更多信息，请参阅 Hyperledger Fabric 文档中的 [Adding an Org to a Channel 教程](https://hyperledger-fabric.readthedocs.io/en/release-1.2/channel_update_tutorial.html)。

以下列表显示了常规步骤，并且这些任务将由联盟中的不同组织集来执行。

1. 每个加入联盟的组织都需要[准备组织定义](/docs/services/blockchain/peer_operate_icp.html#organization-definition)。
2. 通过将组织添加到排序节点系统通道，排序节点组织管理员可[构成联盟](#consortium)。
3. 通过准备通道配置事务，联盟的任何组织都可以[创建新的通道](/docs/services/blockchain/peer_operate_icp.html#peer-icp-channeltx)。

## 获取 Fabric 工具
{: #get-fabric-tools}

您需要下载以下 Hyperledger Fabric 工具来更新系统通道。
- [peer](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/peercommand.html)，用于支持访存起源区块和更新系统通道。
- [configtxlator](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/configtxlator.html)，用于将 protobuf 格式的通道配置转换为更易于读取和更新的 JSON 格式。

1. 确定要存储工具的位置，然后运行以下命令：

  ```
  curl -sSL http://bit.ly/2ysbOFE | bash -s 1.2.1 1.2.1 -d -s
  ```
  {:codeblock}

2. 设置工具的路径，即上述 Fabric 工具的下载位置：

  ```
  export PATH=$PATH:<full_path_to_bin_folder>
  ```
  {:codeblock}

  例如，如果已将二进制文件安装在主目录中，那么应将 `PATH` 设置为：

  ```
  export PATH=$PATH:$HOME/bin
  ```
  {:codeblock}

3. 下载二进制文件将创建包含 core.yaml、orderer.yaml 和 configtx.yaml 文件的 config 文件夹。将 Fabric 配置路径设置为此 config 目录：

  ```
  export FABRIC_CFG_PATH=<full_path_to_config_folder>
  ```
  {:codeblock}

  例如：
  ```
  export FABRIC_CFG_PATH=$HOME/config
  ```
  {:codeblock}

## 创建组织定义
{: #org-definition}

组织的**定义**包含组织名称（MSP 标识）和相关证书。系统通道和应用程序通道将使用此定义在策略中包含组织，策略用于控制通道创建、更新和事务背书。每个想加入联盟的组织都需要完成此步骤。要了解更多信息，请参阅[准备组织定义](/docs/services/blockchain/peer_operate_icp.html#organization-definition)。

## 构成联盟
{: #consortium}

重新调用构成联盟的高级别流程：

1. 排序节点系统通道仅使用作为系统通道成员的排序节点组织构成。该组织可以进行更新，而无需额外签名。
2. 排序节点组织管理员接收来自希望加入联盟的成员的组织定义。排序节点组织使用组织定义来更新排序节点系统通道的配置。

### 获取组织定义

排序节点需要接收来自希望加入联盟的成员的[组织定义](/docs/services/blockchain/peer_operate_icp.html#organization-definition)。这需要在与向您发送 JSON 文件（包含成员的 MSP 标识和加密资料）的其他成员的频带外操作中完成。对于以下命令中的参考信息，我们假定您已创建名为 `org-definitions` 的文件夹，并已将所有相关文件放在该目录中。

### 访存系统通道的起源区块

1. 使用在部署期间建立的排序节点组织 MSP 名称、排序节点 MSP 的路径以及排序节点 TLS CA 证书的路径来设置以下环境变量。

  ```
  export FABRIC_CFG_PATH=<PATH_TO_/config_FOLDER>
  export CORE_PEER_LOCALMSPID=<MSPID_OF_ORDERER_ORG>
  export CORE_PEER_MSPCONFIGPATH=<PATH_TO_ADMIN_MSP>  
  export CORE_PEER_TLS_ROOTCERT_FILE=<PATH_TO_ORDERER_TLS_CERT>
  export CORE_PEER_TLS_ENABLED=true
  export PROXY = <PROXY_IP>
  export ORDERER_PORT = <EXTERNAL_NODE_PORT>
  ```
  {:codeblock}

  将相关字段替换为您自己的信息。

    - 将 `<CORE_PEER_LOCALMSPID>` 替换为排序节点组织的 MSP 标识。还可以通过运行以下命令，并将 `<orderer pod name>` 替换为排序节点的 pod 的值，在排序节点容器中显示此标识：

      ```
      kubectl exec -it <orderer pod name> -c orderer sh
      $ env | grep ORDERER_GENERAL_LOCALMSPID
      ```
      {:codeblock}

      输出可能类似于以下内容：

      ```
      ORDERER_GENERAL_LOCALMSPID=ordererOrg
      ```

      因此，`CORE_PEER_LOCALMSPID` 是 ordererOrg。

    - 将 `<CORE_PEER_MSPCONFIGPATH>` 替换为排序节点组织管理员的 MSP 文件夹的路径。
    - 将 `<CORE_PEER_TLS_ROOTCERT_FILE>` 替换为 TLS CA 证书的路径。
    - 将 `<PROXY_IP>` 替换为[排序节点端点信息](#orderer-endpoint)中的代理 IP 地址。
    - 将 `<EXTERNAL_NODE_PORT>` 替换为[排序节点端点信息](#orderer-endpoint)中的外部节点端口。

  环境变量可能类似于以下示例：

  ```
  export FABRIC_CFG_PATH=$HOME/config
  export CORE_PEER_LOCALMSPID=ordererOrg
  export CORE_PEER_MSPCONFIGPATH=$HOME/fabric-ca-client/orderer-admin/msp/
  export CORE_PEER_TLS_ROOTCERT_FILE=$HOME/fabric-ca-client/orderer-tls/orderertls.pem 
  export CORE_PEER_TLS_ENABLED=true
  export PROXY = 9.30.94.174
  export ORDERER_PORT = 30159
  ```

  您可以在任意点通过发出以下命令来检查这些环境变量：

  ```
  env | grep CORE
  env | grep PATH
  ```
  {:codeblock}

2. 将通道名称设置为环境变量。排序节点系统通道的名称将始终为 `test-system-channel-name`。

  ```
  export CHANNEL_NAME=test-system-channel-name
  ```
  {:codeblock}

3. 访存系统通道的起源区块。首先，在 `org-definitions` 中创建名为 `configupdate` 的目录，用于存储配置更新过程中生成的配置文件。您可以将此目录命名为任意名称，但以下样本命令将使用 `configupdate`。

  ```
  peer channel fetch config ./configupdate/genesis.pb -o $PROXY:$ORDERER_PORT -c $CHANNEL_NAME --tls --cafile <PATH_TO_ORDERER_TLS_CERT>
  ```
  {:codeblock}

  将 `<orderer_TLS_root_cert_file>` 替换为在此[步骤](#orderer-tls)中创建的 `orderertls.pem` 文件的路径。例如，命令可能类似于以下示例：

  ```
  peer channel fetch config ./configupdate/genesis.pb -o $PROXY:$ORDERER_PORT -c $CHANNEL_NAME --tls  --cafile $HOME/fabric-ca-client/orderer-tls/orderertls.pem
  ```
  {:codeblock}

  如果成功，此命令将生成以下输出。

  ```
  2018-10-25 20:53:06.377 UTC [cli/common] readBlock -> INFO 002 Received block: 0
  2018-10-25 20:53:06.380 UTC [cli/common] readBlock -> INFO 003 Received block: 0
  ```

  在此之后，您还可以在文件系统中找到 `genesis.pb` 区块。

### 创建系统通道更新
{: #system-channel-update}

下载的 [Fabric 工具](#get-fabric-tools) `configtxtlator` 用于将 protobuf 格式的通道配置转换为 JSON 格式，反之亦然。

这些步骤遵循有关[将区块转换为 JSON 格式]( https://hyperledger-fabric.readthedocs.io/en/release-1.2/channel_update_tutorial.html#convert-the-configuration-to-json-and-trim-it-down)的通道更新教程的常规流程。您需要对教程中的命令进行一些更改，以反映出您更新的是排序节点系统通道，而不是应用程序通道。您可以访问该教程以了解有关此过程的更多详细信息。本部分仅提供了命令。

1. 将组织定义 JSON 文件从在其中[创建组织](/docs/services/blockchain/peer_operate_icp.html#organization-definition)的文件夹复制到 `configupdate` 文件夹。在以下示例命令中，组织定义 JSON 文件为 `org1definition.json`：

   ```
   cp <path_to_config_folder>/org1definition.json $HOME/fabric-ca-client/org-definitions/configupdate
   ```
   {:codeblock}

2. 将起源区块转换为 JSON 格式。运行以下命令：

  ```
  cd configupdate
  configtxlator proto_decode --input genesis.pb --type common.Block --output ./config_block.json
  jq .data.data[0].payload.data.config ./config_block.json  > config.json
  ```
  {:codeblock}

3. 运行以下命令，将组织的加密资料添加到联盟配置中。将 <NEWORGMSP> 替换为[已创建的组织](/docs/services/blockchain/peer_operate_icp.html#organization-definition)的组织 MSP 标识。

  ```
  jq -s '.[0] * {"channel_group":{"groups":{"Consortiums":{"groups":{"SampleConsortium":{"groups": {"<NEWORGMSP>":.[1]}}}}}}}' config.json ./orgdefinition.json > modified_config.json
  ```
  {:codeblock}

  该命令可能类似于以下示例：

  ```
  jq -s '.[0] * {"channel_group":{"groups":{"Consortiums":{"groups":{"SampleConsortium":{"groups": {"org1":.[1]}}}}}}}' config.json ./orgdefinition.json > modified_config.json
  ```

4. 对需要加入联盟的每个组织重复此命令。确保将 `./orgdefinition.json` 文件更改为新的组织 JSON 文件。

  ```
  jq -s '.[0] * {"channel_group":{"groups":{"Consortiums":{"groups":{"SampleConsortium":{"groups": {"org1":.[1]}}}}}}}' config.json ./orgdefinition.json > modified_config.json
  ```
  {:codeblock}

5. 运行以下命令，将 `modified_config.json` 文件转换为 JSON 格式的配置更新。

  ```
  configtxlator proto_encode --input config.json --type common.Config --output config.pb 
  configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb 
  configtxlator compute_update --channel_id $CHANNEL_NAME --original config.pb --updated modified_config.pb --output config_update.pb 
  configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate --output config_update.json
  ```
  {:codeblock}

6. 完成这些命令后，将 `config_update` 装入信封。

  ```
  echo '{"payload":{"header":{"channel_header":{"channel_id":"'$CHANNEL_NAME'", "type":2}},"data":{"config_update":'$(cat config_update.json)'}}}' | jq . > config_update_in_envelope.json
  ```
  {:codeblock}

7. 运行以下命令，将配置更新转换回 protobuf 格式：

  ```
  configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope --output ./config_update_in_envelope.pb
  ```
  {:codeblock}

### 将更新发送到系统通道

完成这些步骤之前，请确保已正确设置 `FABRIC_CFG_PATH`、`$PROXY` 和 `ORDERER_PORT`。运行以下命令：

```
export ORDERER_CA=<path and file name of the orderer TLS CA cert>
export PROXY = <proxy ip address from [orderer endpoint information](#orderer-endpoint)>
export ORDERER_PORT = <external node port from [orderer endpoint information](#orderer-endpoint)>
```
{:codeblock}

环境变量应该类似于以下示例：

```
ORDERER_CA = /Users/chandra/fabric-ca-client/orderer-tls/orderertls.pem
CORE_PEER_TLS_ROOTCERT_FILE=/Users/chandra/fabric-ca-client/orderer-tls/orderertls.pem
CORE_PEER_LOCALMSPID=ordererOrg
CORE_PEER_TLS_ENABLED=true
CORE_PEER_MSPCONFIGPATH=/Users/chandra/fabric-ca-client/orderer-admin/msp
PROXY = 9.30.94.174
ORDERER_PORT = 30159

```

创建通道配置更新（即 `config_update_in_envelope.json`）后，排序节点组织可以使用以下命令来提交通道更新：

```
peer channel update -f config_update_in_envelope.pb -c $CHANNEL_NAME -o $PROXY:$ORDERER_PORT --tls --cafile $ORDERER_CA
```
{:codeblock}

此命令同时对更新请求签名并将其发送到排序节点。由于排序节点组织是系统通道的唯一管理员，因此该更新只需要一个签名就能成为有效请求。如果系统通道更新成功，您将看到以下输出：

```
2018-10-28 23:44:00.498 UTC [channelCmd] update -> INFO 002 Successfully submitted channel update
```

可以在单个排序节点系统通道配置更新中包含多个组织定义，但最好是一次对一个组织更新该通道，并检查以确保更新成功。

## 查看排序节点日志
{: #orderer-view-logs}

在命令行中使用 [`kubectl CLI 命令`](#ca-kubectl-configure)或通过 ICP 集群中包含的 [Kibana ![外部链接图标](../images/external_link.svg "外部链接图")](https://www.elastic.co/products/kibana "您使用 Elasticsearch 的窗口")，可以查看组件日志。

- 使用 `kubectl logs` 命令来查看 pod 内的容器日志。如果不确定 pod 名称，请运行以下命令来查看 pod 的列表。

  ```
  kubectl get pods
  ```
  {:codeblock}

  然后，运行以下命令并将 `<pod_name>` 替换为上面命令输出中 pod 的名称，以检索位于 pod 内的排序节点容器的日志：

  ```
  kubectl logs <pod_name> -c orderer
  ```
  {:codeblock}

  有关 `kubectl logs` 命令的更多信息，请参阅 [Kubernetes 文档 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs “Getting Started”)

- 或者，可以使用 [ICP 集群管理控制台](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/troubleshoot/events.html)来访问日志，这将在 Kibana 中打开日志。

  **注：**在 Kibana 中查看日志时，可能会收到响应 `No results found`。如果 ICP 将工作程序节点 IP 地址作为其主机名，那么会发生此情况。要解决此问题，请在面板顶部除去以 `node.hostname.keyword` 开头的过滤器，随后相关日志即可显示。

## 故障诊断
{: #orderer-troubleshooting}

### **问题：**`peer channel update` 命令失败，并返回错误。
{: #orderer-peer-channel-update-error}

运行 `peer channel update` 命令时，可能会收到以下错误：

```
INFO 001 Endorser and orderer connections initialized Error: got unexpected status: BAD_REQUEST -- initializing channelconfig failed: could not create channel Consortiums sub-group config: Attempted to define two different versions of MSP: Org1
```

同级使用的组织 MSP 标识与排序节点的相同时，会导致此错误。  

**解决方案：**  

要解决此问题，您需要重新部署同级，并指定不同于部署排序节点时所使用标识的组织 MSP 标识。
