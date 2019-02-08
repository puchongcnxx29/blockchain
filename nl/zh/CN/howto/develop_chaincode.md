---

copyright:
  years: 2017, 2018
lastupdated: "2018-12-07"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}

# 编写智能合同

***[此页面是否有用？请告诉我们。](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

链代码（也称为智能合同）是允许您在区块链分类帐上读取和更新数据的软件。链代码可以将业务逻辑转变为区块链网络的所有成员同意和验证的可执行程序。业务逻辑包括在各方之间交换的资产的定义。它还包含执行事务所需的条款和条件。通过将这些规则转变为区块链上的代码，企业可以简化业务处理和审计，并减少大量手动处理和书面作业。

例如，假设汽车经销商、保险公司和政府监管机构的网络决定使用区块链来跟踪车辆的所有权。链代码可能要求所有车辆都具有有效的注册和车辆标识号，才能添加到网络。当车辆售出时，该链代码可能要求代管资金，直至该车辆由监管机构注册给新车主。当新注册完成时，将记录新车主并自动转移资金。

以下教程将引导您完成构建链代码的基本方面，包括：

- [如何开始编写链代码](#write-chaincode)
- [链代码和数据之间的关系](#install-chaincode)
- [跨链代码事务](#cross-chaincode-transactions)

本教程还介绍了 Fabric 中可通过链代码访问的重要方面：

- [专用数据](#private-data)
- [将索引用于 couchDB](#indexes)

## 编写链代码
{: #write-chaincode}

链代码可以使用多种语言编写，并且 {{site.data.keyword.blockchainfull_notm}} Platform 支持使用 Go 和 Node.js 编写的链代码。链代码允许用户使用 Fabric Chaincode 接口提供的 API 来查询和更改存储在区块链中的数据。区块链上的数据存储在通道[分类帐 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/ledger/ledger.html "分类帐") 的全局状态键值对中。链代码使用 get 命令来检索值并使用 put 命令来创建或更新值。通过使用这些基本操作，您可以构建用于定义网络业务规则的函数。这些函数可以由应用程序调用，并显示给网络的最终用户。继续以车辆网络为例，您可以创建一个函数，用于仅允许汽车经销商在提供了有效车辆标识号时使用 put 命令将新的车辆添加到分类帐。

您可以通过访问 Hyperledger Fabric 社区文档中[开发人员链代码教程 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html "开发人员链代码教程") 来了解如何开始编写链代码。本教程将指导您构造简单的链代码，该链代码创建和读取资产，并向您介绍流程中使用了哪些 API。您还可以找到所有链代码语言的链代码 API 参考指南。
[Fabric 样本存储库 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://github.com/hyperledger/fabric-samples "Fabric 样本") 的链代码文件
夹中还有更多示例。

## 安装链代码
{: #install-chaincode}

由于链代码提供了通道上事务的结构，因此需要将链代码安装在加入了该通道并且要使用该链代码更新或查询通道分类帐的所有同级上。然后，通道的一个成员可以实例化通道上的链代码，并设置链代码的背书策略。
可以使用“网络监视器 UI”、Fabric Peer 命令行界面或者从使用 [Fabric SDK](/docs/services/blockchain/v10_application.html#operate-sdk) 的客户机应用程序执行链代码的安装和实例化。要了解如何使用“网络监视器 UI”部署链代码，请参阅[安装、实例化和更新链代码](/docs/services/blockchain/install_instantiate_chaincode.html)。

## 链代码和数据
{: #chaincode-data}

每个通道只有一个分类帐，并且该分类帐上的数据按唯一键和将键值对添加到分类帐的链代码进行分区。成员只能使用正确的键和关联的链代码在通道分类帐上读取或更新数据。链代码可以访问的数据称为该链代码的名称空间，并且分类帐上的所有数据都在一个链代码的名称空间内。链代码要与其名称空间之外的数据进行交互，只能使用针对该链代码的可访问相关数据的[跨链代码事务](#cross-chaincode-transactions)。

我们可以将 Fabric 样本存储库中的 fabcar 链代码用作链代码如何与数据交互的示例。实例化智能合同时，将创建名称空间。某些链代码在实例化时接受一组键值对参数，并使用该数据来初始化名称空间。Fabric 样本存储库中的 fabcar 链代码在实例化时不接受任何参数。对于 fabcar，您需要使用 initLedger 或 createCar 函数将数据添加到名称空间。例如，传递自变量 `{"Args":["createCar",'CAR1', 'Honda', 'Accord', 'Black', 'Tom']}` 将使 `{Key='CAR1', value={'Honda', 'Accord', 'Black', 'Tom'}}` 键值对成为 fabcar 名称空间。

只有 fabcar 链代码才能更改此数据。假定用户希望通过使用下面的 changeCarOwner 函数将汽车转移给新的车主：
```
func (s *SmartContract) changeCarOwner(APIstub shim.ChaincodeStubInterface, args []string) sc.Response {

	if len(args) != 2 {
		return shim.Error("Incorrect number of arguments. Expecting 2")
	}

	carAsBytes, _ := APIstub.GetState(args[0])
	car := Car{}

	json.Unmarshal(carAsBytes, &car)
	car.Owner = args[1]

	carAsBytes, _ = json.Marshal(car)
	APIstub.PutState(args[0], carAsBytes)

	return shim.Success(nil)
}
```
{:codeblock}

您可以使用以下自变量来调用链代码：
```
{"Args":["changeCarOwner",'CAR1','Mark']}
```
{:codeblock}

然后，链代码使用 get 命令从链代码名称空间生成读集。它使用 put 命令来创建事务写集。链代码调用还会创建唯一的事务标识。
```
TxId = Txid1
Read Set: `{Key='CAR1', value[3]='Tom'}`
Write Set: `{Key='CAR1', value[3]='Mark'}`
```
{:codeblock}

每个安装有链代码的背书同级都将使用读集和写集模拟事务。如果每个同级的模拟是一致的，那么这些事务将视为有效，并且可落实到分类帐。事务完成后，fabcar 名称空间将变为 `{Key='CAR1', value='Honda', 'Accord', 'Black', 'Mark'}`。请注意，车主已从 Tom 更改为 Mark。

链代码和数据之间的 1 对 1 关系为开发网络和应用程序带来了很重要的考虑事项。链代码软件应通过安装名称相同、版本不同的链代码来更新，而不是使用新的链代码来更新。此更新过程将保留链代码名称空间，并允许您继续使用现有数据。您还可以使用不同的链代码来限制对敏感数据的访问，或者以并行方式读写数据。

## 跨链代码事务
{: #cross-chaincode-transactions}

您可以使用链代码来调用其他链代码。这允许链代码查询和写入其名称空间之外的数据。链代码可以使用在同一通道上实例化的链代码读取和更新数据。但是，它们只能使用不同通道上的链代码来查询数据。调用原始链代码的组织需要经过授权才可以调用作为事务目标的链代码。

例如，您可以使用下面的 `crossChaincodeChangeCarOwner` 函数从其他链代码中调用 fabcar。由于此函数通过更改车主来更新数据，因此需要调用同一通道上实例化的 fabcar 链代码。
```
func (s *SmartContract) crossChaincodeChangeCarOwner(APIstub shim.ChaincodeStubInterface, args []string) sc.Response {
	newOwnerArgs := toChaincodeArgs("changeCarOwner", args[0], args[1])

	chaincodeName := "fabcar"
	channelName := "defaultchannel"

	APIstub.InvokeChaincode(chaincodeName, newOwnerArgs, channelName)

	return shim.Success(nil)
}
```
{:codeblock}

如果使用此代码创建新合同并调用函数 `crossChaincodeChangeCarOwner`，那么事务将生成新的事务标识和写集。
```
Contract: newContract
TxId = Txid2
Write Set: `{Key='CAR1', value[3]='Mark'}`
```
{:codeblock}

此 TXid 和写集将传递到 fabcar 链代码。
```
Contract: fabcar
TxId = Txid2
Read Set: `{Key='CAR1', value[3]='Mark'}`
Write Set: `{Key='CAR1', value[3]='Joe'}`
```
{:codeblock}

由于 `fabcar` 与 `newContract` 位于同一通道上，因此允许 `crossChaincodeChangeCarOwner` 函数写入 `fabar` 名称空间。新的 `fabcar` 名称空间将为 `{Key='CAR1', value='Honda', 'Accord', 'Black', 'Joe'}`。

## 专用数据
{: #private-data}

专用数据集合是 Hyperledger Fabric 网络（V1.2 或更高版本）的功能。多个组织集合使用通道进行事务处理，而不向其他组织公开数据。但是，组织可能希望将敏感信息保持为专用，而不向通道的其他成员公开数据。
在此情况下，他们可以使用专用数据集合来将信息仅存储在需要该数据的组织的同级上。例如，批发商和农场可能希望使用与其他农场相同的通道以提供销售证明。但是，他们可以使用专用集合将销售的敏感方面（例如价格）保持为专用。要了解使用区块链中专用数据的更多信息，请访问 Fabric 文档中的[专用数据 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html "专用数据") 概念文章。

只要网络位于 Fabric V1.2 或更高版本，就可以在 {{site.data.keyword.blockchainfull_notm}} Platform 上使用专用数据。您可以将专用数据集合配置文件添加到链代码，然后再将其安装到同级上。接着，您可以使用特定于专用数据的链代码 API 来输入数据并从集合中检索数据。

有关如何将专用数据集合用于链代码的更多信息，请参阅 Fabric 文档中的[使用专用数据 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/latest/private_data_tutorial.html "使用专用数据")。您无法使用“网络监视器”安装带有集合配置文件的链代码。但是，您可以使用 Fabric SDK 来安装、实例化或更新使用专用数据的链代码。有关更多信息，请参阅 Node SDK 文档中的[如何使用专用数据 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://fabric-sdk-node.github.io/release-1.3/tutorial-private-data.html "如何使用专用数据")。**请注意**，您需要将 SDK 的 signCert 上传到 {{site.data.keyword.blockchainfull_notm}} Platform，然后您才可以使用 SDK 安装和实例化同级上的链代码。有关如何使用 Fabric SDK 与网络交互并操作网络的更多信息，请参阅[开发应用程序](../v10_application.html)和[使用 SDK 操作网络](../v10_application.html#operate-sdk)。


## 将索引用于 CouchDB
{: #indexes}

如果分类帐数据存储在 CouchDB 中，那么强烈建议您为 CouchDB 查询创建索引，并在链代码中使用这些索引。索引允许应用程序在网络添加更多全局状态的事务和条目块时高效地检索数据。另外，CoupchDB 允许您根据通道分类帐上的数据从链代码执行丰富的数据查询。

有关 CouchDB 以及如何设置索引的更多信息，请参阅 Hyperledger Fabric 文档中的 [CouchDB 作为状态数据库 ![外部链接图标](../images/external_link.svg "外部链接图标")](http://hyperledger-fabric.readthedocs.io/en/release-1.1/couchdb_as_state_database.html "CouchDB 作为状态数据库"){:new_window}。您还可以在 [Fabric CouchDB 教程 ![外部链接图标](../images/external_link.svg "外部链接图标")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/couchdb_tutorial.html)
中找到将索引用于链代码的示例。请访问“开发应用程序”教程中的[使用 CouchDB 时的最佳实践](/docs/services/blockchain/v10_application.html#couchdb-indices)，以获取有关如何从应用程序查询数据的更多信息。
