---

copyright:
  years: 2017, 2018
lastupdated: "2018-08-31"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# 使用 Swagger API 创建或加入网络


***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


{{site.data.keyword.blockchainfull}} Platform 公开了大量可用于在 {{site.data.keyword.cloud_notm}} 上创建或加入区块链网络的 REST API。您可以使用与网络相关联的 [Swagger UI](/docs/services/blockchain/swagger_apis.html) 来试用这些 API。
{:shortdesc}


## 检索 API 的基本认证凭证
{: #retrieve-id-token}

开始之前，需要在 {{site.data.keyword.cloud_notm}} 中使用入门套餐或企业套餐来创建 [{{site.data.keyword.blockchainfull_notm}} Platform 服务实例 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://console.bluemix.net/catalog/services/blockchain)。

要使用 Swagger API 来创建或加入网络，需要基本认证凭证以确保您有权访问 {{site.data.keyword.cloud_notm}} 中的服务实例。

1. 在 [{{site.data.keyword.cloud_notm}} 仪表板 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://console.bluemix.net/dashboard/apps/) 中，打开创建的服务实例。
2. 单击左侧导航器中的**服务凭证**。
3. 单击**服务凭证**页面上的“新建凭证”按钮以创建新凭证。
    1. 为凭证提供名称，例如 *CreateJoin*。
    2. 在“添加内联配置参数”字段中，输入 **{"type": "service_instance_token"}**。
    3. 单击**添加**按钮。
    ![检索服务凭证](../images/service_credentials.gif "检索服务凭证")
4. 创建新凭证后，单击此凭证的**操作**标题下方的**查看凭证**。凭证的内容类似于以下示例：

    ```
    {
      "service_instance_id": "60cc757d-1234-7866-9893-491cec2d0eaa",
      "service_instance_token": "T8-W0rHv_vEya63P8KcVUwxAXXbDmihGcDRD5AcHVXGn6S2jbxx2LikPz8w26c3k",
      "description": "This token can be used with the IBP APIs to create or join a network. It can only be used once."
    }
    ```
    {:codeblock}

    `service_instance_id` 充当基本认证用户标识，而 `service_instance_token` 充当基本认证密码。在调用**创建网络**或**加入网络** API 时，使用这些值作为基本认证凭证。

    如果在执行前述步骤以创建服务凭证之前创建了区块链网络，那么凭证的内容还将包含网络信息，类似于以下示例：

    ```
    {
      "PeerOrg1": {
        "url": "https://ibmblockchain_xyz.ng.bluemix.net",
        "network_id": "92d511f7e587413c8a9848fdae595ef2",
        "key": "PeerOrg1",
        "secret": "T8eUA65l-qtznUHL10KzQ7IK-3BVWWfHu5-hpCiDdXCRQyNfeyIm1p5NT7g17l6U"
      }
    }
    ```
    {:codeblock}

    **注**：对于**邀请** API，`key` 充当基本认证用户标识，而 `secret` 充当基本认证密码。


## 检查可用网络位置
{: #check-location}

您可以使用 API 以仅在可用网络位置中创建区块链网络。在创建网络之前，请使用以下 API 来检查当前可用网络位置列表。运行此 API 不需要凭证。

```
https://ibmblockchain-v2.ng.bluemix.net/api/v1/network-locations/available
```
{:codeblock}

返回的可用网络位置列表类似于以下内容：

```
{
  "DAL": {
    "location_id": "DAL",
    "description": "Dallas",
    "status": "available"
    "swagger_url": "https://ibmblockchain-v2-dal.4.secure.blockchain.ibm.com/api-docs"
  },
  "FFT": {
    "location_id": "FFT",
    "description": "Frankfurt",
    "status": "available"
    "swagger_url": "https://ibmblockchain-v2-fft.2.secure.blockchain.ibm.com/api-docs"
  },
  "TOR": {
    "location_id": "TOR",
    "description": "Toronto",
    "status": "available"
    "swagger_url": "https://ibmblockchain-v2-tor.1.secure.blockchain.ibm.com/api-docs"
  }
}
```
{:codeblock}

如果打算创建网络，请从 API 返回的列表中选择要在其中创建网络的位置。请注意，``location_id`` 和 ``swagger_url`` 与该位置相关联。  

如果打算加入网络，请记录与邀请电子邮件中指定的 ``location_id`` 关联的 ``swagger_url``。

``swagger_url`` 表示在使用以下 API 创建或加入网络时将使用的 API 端点。


## 创建网络

**注**：如果使用入门套餐，那么无需使用 API 创建网络，因为在 {{site.data.keyword.cloud_notm}} 中创建区块链服务实例时提供缺省网络。

如果使用企业套餐，那么需要完成两个步骤以使用 API 创建网络。

1. 在 {{site.data.keyword.cloud_notm}} 上使用企业套餐创建区块链服务实例<!-- or Enterprise Plus Plan-->。检索服务实例标识和令牌作为基本认证用户名和密码。有关更多信息，请参阅[检索 API 的基本认证凭证](#retrieve-id-token)。

2. 使用这些服务凭证来调用**创建网络** API。针对从[检查可用网络位置](#check-location)中检索到的 API ``swagger_url`` 发出此 API。浏览至 ``swagger_url`` 链接以使用 Swagger UI 发出“创建网络 API”，或者通过编程方式使用不含 ``/api-docs`` 的 URL 地址来发出该命令。例如，

    ```
https://ibmblockchain-v2-tor.1.secure.blockchain.ibm.com/api/v1/networks
    ```
    {:codeblock}

**参数**：
- `location_id`：可用网络位置的标识。指定在[检查可用网络位置](#check-location)中记下的 `location_id` 的值。
- `company_name`：作为网络中成员的标识。
- `email`：用于接收通知的电子邮件地址。
- `peers`：要针对此成员创建的同级的数量。有效值为 0 - 6。您还可以稍后在“网络监视器 UI”中为成员创建同级。
- `ledger_type`：此网络的分类帐的类型。有效值为 levelDB 和 couchDB。**levelDB** 是缺省值。


## 邀请新成员加入网络

创建区块链网络后，可以邀请其他成员加入您的网络。需要指定要邀请新成员加入的网络的标识。邀请成员所需的基本认证凭证与在**创建网络** API 中使用的凭证不同。<!--In order to get the basic auth information you will need to follow the same steps in "Retrieving basic auth information for API". --> 您可以在 [Swagger UI ](swagger_apis.html#retrieving-network-credentials) 上使用**检索网络凭证 ** API 或者从 {{site.data.keyword.cloud_notm}} 中的服务实例[检索 API 的基本认证信息](#retrieve-id-token)来获取用于邀请成员的凭证。

```
/networks/{networkID}/invite
```
{:codeblock}

**参数**：
- `email`：要邀请加入网络的成员的电子邮件地址。
- `company_name`：为成员定义的用于加入网络的标识。受邀请的成员可在加入网络时更改其标识。

受邀请的成员将收到电子邮件邀请，其中包含有关如何加入您的网络的指示信息。


## 加入网络

如果您获邀加入区块链网络，那么您将收到包含 `location_id` 和 `network id` 的网络邀请电子邮件。

1. 在加入网络之前，您需要创建 {{site.data.keyword.blockchainfull_notm}} Platform 服务实例，并检索服务实例标识和令牌作为基本认证用户名和密码。有关更多信息，请参阅[检索 API 的基本认证信息](#retrieve-id-token)。

2. [检查可用网络位置](#check-location)以获取邀请电子邮件中 `location_id` 的 `swagger_url`。此 URL 将类似于以下内容：

    ```
https://ibmblockchain-v2-tor.1.secure.blockchain.ibm.com/api-docs
    ```
    {:codeblock}

3. 浏览至 `swagger_url` 以使用 Swagger UI 发出“加入 API”，或者通过编程方式使用 `swagger_url` 来提交“加入”请求。将 `/api-docs` 替换为 `/api/v1/networks/[network_id]]/join`，并使用邀请电子邮件中的值填写 `network_id`。生成的 URL 将类似于以下内容：

    ```
https://ibmblockchain-v2-tor.1.secure.blockchain.ibm.com/api/v1/networks/56102acee0e4487889ef09db681bada0/join
    ```
    {:codeblock}

    **注**：对于**加入** API，请使用在步骤 1 中检索到的服务实例标识和令牌作为基本认证用户名和密码。

**参数**：
- `company_name`：作为网络中成员的标识。这将替换邀请者指定的名称。
- `email`：用于接收通知的电子邮件地址。应该与邀请通知中的电子邮件地址相匹配。
- `peers`：要针对此成员创建的同级的数量。有效值为 0 到 6。您还可以稍后在“网络监视器”中为成员创建同级。
