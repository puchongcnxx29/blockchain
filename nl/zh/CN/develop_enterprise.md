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
{:tip: .tip}

# 在企业套餐上部署业务网络
{: #deploying-a-business-network}


***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


{{site.data.keyword.blockchainfull}} Platform: Develop 的开发者工具可帮助您创建**业务网络定义**，然后可将其打包成业务网络归档 (`.bna`)。开发者环境支持您将 `.bna` 文件部署到本地或云 {{site.data.keyword.blockchain}} 以进行开发和共享。

本教程将讨论业务网络生命周期的下一步，即通过将 `.bna` 部署到 {{site.data.keyword.blockchainfull_notm}} Platform 企业套餐来激活业务网络。

## 开始之前

确保已安装 {{site.data.keyword.blockchainfull_notm}} 开发者环境，并已熟悉开发和部署业务网络。[Hyperledger Composer 文档](https://hyperledger.github.io/composer/latest/business-network/business-network-index)中提供了有关编写业务网络的指导信息。

您需要访问 {{site.data.keyword.blockchainfull_notm}} Platform 的企业套餐实例，并预先创建您的同级。有关 {{site.data.keyword.blockchainfull_notm}} Platform 企业套餐的更多信息，请参阅[企业套餐概述](/docs/services/blockchain//enterprise_plan.html)。

## 步骤 1：为 {{site.data.keyword.blockchainfull_notm}} Platform 创建连接概要文件

1. 创建用于存储连接详细信息的目录，例如：

    ```
    /Users/myUserId/.composer-connection-profiles/bmx-hlfv11
    ```
    {:codeblock}

    每个连接概要文件都应该包含 `connection.json` 文件。在 `.composer-connection-profiles` 下创建一个新目录，在本例中为 `bmx-hlfv11`。这将是您在使用 Hyperledger Composer 和 {{site.data.keyword.blockchainfull_notm}} Platform 时要使用的概要文件的名称。

    ```
    mkdir -p ~/.composer-connection-profiles/bmx-hlfv11
    cd ~/.composer-connection-profiles/bmx-hlfv11
    ```
    {:codeblock}

    现在，您应该具有以下目录结构：

    ```
    /Users/myUserId/.composer-connection-profiles/bmx-hlfv11
    ```

## 步骤 2：检索连接概要文件

1. 从 {{site.data.keyword.blockchainfull_notm}} Platform 仪表板，选择**概述**，然后选择**连接概要文件**和**下载**按钮以检索连接概要文件。

2. 将连接概要文件保存到上一步中所创建的目录结构中。将其命名为 **connection.json**。

## 步骤 3：添加通道信息

1. 在 `connection.json` 中添加通道值，以匹配您计划创建和部署业务网络的通道的名称。在提供的示例连接概要文件模板中，通道元素如下所示：`"channels": {},`。

## 步骤 4：准备同级

**请注意**：**必须**在创建要将业务网络部署到的通道之前执行此步骤。如果在通道创建后执行此步骤，部署的业务网络**可能不会启动**。

在“连接概要文件”文档中，**certificateAuthorities** 下是 **registrar** 属性，包含 **enrollId** 和 **enrollSecret** 属性，格式如下：

 ```
"registrar": [
            {
                "enrollId": "admin",
                "enrollSecret": "PA55W0RD12"
            }
        ],
 ```

1. 使用以下命令为 CA 创建一个业务网络卡：

    ```
    composer card create -f ca.card -p bmx-hlfv11 -u admin -s PA55W0RD12
    ```
    {:codeblock}

  确保您正在连接概要文件所在的同一目录中运行该命令。

2. 使用以下命令来导入业务网络卡：

    ```
   composer card import -f ca.card -c ca
   ```
    {:codeblock}

3. 导入卡后，可以将其用于通过以下命令从认证中心获取证书：

    ```
    composer identity request --card ca --path ./credentials -u admin -s PA55W0RD12
    ```
    {:codeblock}

    `composer identity request` 命令会创建包含相关证书文件的凭证目录。

4. 从导航菜单中选择**成员**，然后选择**证书**菜单选项，并单击**添加证书**按钮。

5. 在**名称**字段中输入此证书的唯一名称，此名称不能包含破折号或连字符。

6. 打开早些时候创建的 `admin-pub.pem` 文件，将此文件的内容复制到 **Key** 字段中，然后按**提交**按钮。

7. 使用用户界面停止同级，然后启动同级。

8. 现在，该证书应该显示在证书列表中。

## 步骤 5：创建您的通道

1. 从左侧面板上的导航菜单中选择**通道**，然后单击**新建通道**按钮。

2. 输入“通道名称”和可选的描述，然后单击**下一步**。请注意，通道名称必须与您在连接概要文件中为 channel 属性指定的名称相匹配。

3. 根据需要授予许可权，然后单击**下一步**。

4. 选择需要接受通道更新并提交请求的操作员数量的策略。

5. 您现在应该转至**通知**部分，在其中应该有新的请求要复查。单击**复查请求**按钮。

6. 单击**接受**按钮后，您将重新转至**通知**部分，在其中可以看到现在可以提交该请求。单击**提交请求**按钮以打开提交对话框，然后单击**提交**按钮。新的通道已创建。

7. 从导航菜单中选择**通道**。新通道应该位于通道列表中，并且应该具有“尚未添加同级”。单击其旁边的“操作”菜单，并选择**加入同级**。选中要添加的同级旁边的复选框，然后按**添加所选项**。

## 步骤 6：创建新身份以管理业务网络

使用所请求的证书创建业务网络卡。此业务网络卡将有权将链代码安装到具有已上传公共证书的同级，并且这些同级将是认证中心的签发者。

1. 要创建卡，请运行以下命令：

    ```
    composer card create -f adminCard.card -p bmx-hlfv11 -u admin -c ./credentials/admin-pub.pem -k ./credentials/admin-priv.pem --role PeerAdmin --role ChannelAdmin
    ```
    {:codeblock}

    其中，`bmx-hlfv11` 是先前创建的概要文件的名称。

2. 创建卡后，使用以下命令将其导入：

    ```
   composer card import -f adminCard.card -c adminCard
   ```
    {:codeblock}

    现在我们已准备好将 `.bna` 文件部署到 {{site.data.keyword.blockchainfull_notm}} Platform。


## 步骤 7：部署业务网络

现在，您可以将 `.bna` 文件部署到 {{site.data.keyword.blockchainfull_notm}} Platform。

1. 要使用先前步骤中创建的卡，您需要首先安装业务网络，然后将其启动。使用以下命令安装业务网络：

   ```
   composer network install -c adminCard -a myNetwork.bna
   ```
   {:codeblock}

2. 安装业务网络后，使用以下命令来启动业务网络：

    ```
    composer network start -c adminCard -n myNetwork -V networkVersion -A admin -C ./credentials/admin-pub.pem -f delete_me.card
    ```
    {:codeblock}

3. 要检查是否正确部署了业务网络，请创建可用于对网络执行 ping 操作的身份和关联的业务网络卡。

    ```
    composer card create -f admin.card -p bmx-hlfv11 -u admin -n myNetwork -c ./credentials/admin-pub.pem -k ./credentials/admin-priv.pem
    ```
    {:codeblock}

4. 使用以下命令来导入业务网络卡：

    ```
    composer card import -f admin.card
    ```
    {:codeblock}

5. 对网络执行 ping 操作以检查网络是否正在运行：

    ```
    composer network ping -c admin@myNetwork
    ```
    {:codeblock}
