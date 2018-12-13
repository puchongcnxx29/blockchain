---

copyright:
  years: 2017, 2018
lastupdated: "2018-12-08"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}

# Developing business networks
{: #develop-the-network}

**IBM recommends using Hyperledger Composer solely for demos and proof-of-concepts. IBM does not provide support for networks using Hyperledger Composer in production, including the Composer CLI, JavaScript APIs, REST server, and Web Playground.**

***[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


Developing blockchain solutions by using the {{site.data.keyword.blockchainfull}} Platform leverages the Hyperledger Composer open source development toolset. Hyperledger Composer uses a bespoke modeling language, which is combined with JavaScript transactions and access control rules to model a blockchain business network in full _before_ you deploy anything to a real blockchain network.
{:shortdesc}

**Note:** Starter Plan provides sample applications that you can learn and try. You can run sample applications in Starter Plan before you develop your customized business network. For more information, see [About Starter Plan](starter_plan.html).

This tutorial goes through the development of a {{site.data.keyword.blockchain}} solution that begins with creating and modeling a business network, and then deploys it to a running blockchain network.

The following steps outline the basic development of a {{site.data.keyword.blockchain}} solution that is hosted on  {{site.data.keyword.blockchainfull_notm}} Platform:

1. Create the data structure to contain the business network definition. The business network is defined in terms of assets, participants, and transactions by using the [Modelling Language](https://hyperledger.github.io/composer/latest/reference/cto_language).

2. Model the business network by defining the asset classes, participant classes, transaction classes, event classes, and access control rules that will be used after deployment.

3. Package the defined business network into a single deployable file.

4. Deploy the packaged business network to the {{site.data.keyword.blockchainfull_notm}} Platform. Once the business network is deployed, a REST server can be generated to allow integration with external applications.

## Before you begin

Ensure that you have installed the [{{site.data.keyword.blockchainfull_notm}}: Develop development tools](./develop_install.html).

## Step One: Creating the business network structure

A key concept for developing {{site.data.keyword.blockchain}} solutions is the **business network definition (BND)**. The BND defines the data model, transaction logic, and access control rules for your blockchain solution. To create a BND,  you need to create a suitable project structure on disk.

The easiest way to get started is to use the Yeoman generator to create a skeleton business network. The Yeoman generator creates a directory that contains all of the components of a business network, which can be modified for your specific use-case.

1. Create a new directory for your project and navigate to it on the command line.

2. From your new directory, use Yeoman to create a skeleton business network. The following command requires a business network name, description, author name, author email address, license selection, and namespace:
    ```
    yo hyperledger-composer:businessnetwork
    ```
    {:codeblock}

3. Enter `tutorial-network` for the network name, and desired information for description, author name, and author email.

4. Select `Apache-2.0` as the license.

5. Select `org.acme.biznet` as the namespace.

## Step Two: Defining the business network

A business network is made up of assets, participants, transactions, access control rules, and optionally events and queries. The skeleton business network that is created in step one includes a model file (`.cto`), which contains the class definitions for all assets, participants, and transactions in the business network. The skeleton business network also contains an access control document (`permissions.acl`) with basic access control rules, a script file (`logic.js`) that contains transaction processor functions, and a package file (`package.json`) that contains business network metadata.

### Modelling assets, participants, and transactions

The first document to update is the model file (`.cto`). The model file is written using the [Hyperledger Composer Modelling Language](https://hyperledger.github.io/composer/latest/reference/cto_language). The model file contains the definitions of each class of asset, transaction, participant, and event. It implicitly extends the system model that is described in the modelling language documentation.

1. Open the `org.acme.biznet.cto` model file in an editor of your choice. It is located in the `models` folder of the business network you created in the last step.

2. Replace the contents with the following information:

    ```
        /**
         * My commodity trading network
         */
        namespace org.acme.biznet
        asset Commodity identified by tradingSymbol {
            o String tradingSymbol
            o String description
            o String mainExchange
            o Double quantity
            --> Trader owner
        }
        participant Trader identified by tradeId {
            o String tradeId
            o String firstName
            o String lastName
        }
        transaction Trade {
            --> Commodity commodity
            --> Trader newOwner
        }
    ```
    {:codeblock}

3. Save your changes to the `org.acme.biznet.cto` file.


### Adding JavaScript transaction logic

In the model file, a `Trade` transaction was defined, which specifies a relationship to an asset and a participant. The transaction processor function file contains the JavaScript logic to execute the transactions that are defined in the model file.

The `Trade` transaction is intended to simply accept the identifier of the `Commodity` asset that is being traded, and the identifier of the `Trader` participant to set as the new owner.

1. Open the `logic.js` script file in the `lib` directory.

2. Replace the contents with the following information:

    ```
        /**
         * Track the trade of a commodity from one trader to another
         * @param {org.acme.biznet.Trade} trade - the trade to be processed
         * @transaction
         */
        function tradeCommodity(trade) {
            trade.commodity.owner = trade.newOwner;
            return getAssetRegistry('org.acme.biznet.Commodity')
                .then(function (assetRegistry) {
                    return assetRegistry.update(trade.commodity);
                });
        }
    ```
    {:codeblock}

3. Save your changes to `logic.js`.

### Adding access control

Access control is a key part of a business network. All business network must have an access control file that is called `permissions.acl`. Whenever any process is invoked in a deployed business network, the access control list is checked to ensure that the invoking identity or participant can invoke the operation.

For the purposes of this tutorial, set up a simple ACL. Note that this should not be deployed to a production environment.

1. Create a `permissions.acl` file in the `tutorial-network` directory.

2. Add the following access control rules to `permissions.acl`:

    ```
        /**
         * Access control rules for tutorial-network
         */
        rule Default {
            description: "Allow all participants access to all resources"
            participant: "ANY"
            operation: ALL
            resource: "org.acme.biznet.*"
            action: ALLOW
        }

        rule SystemACL {
          description:  "System ACL to permit all access"
          participant: "ANY"
          operation: ALL
          resource: "org.hyperledger.composer.system.**"
          action: ALLOW
        }
    ```
    {:codeblock}

3. Save your changes to `permissions.acl`.

## Step Three: Packaging the business network

After the business network is defined, it must be packaged into a deployable business network archive file (`.bna`).

1. Using the command line, navigate to the `tutorial-network` directory.

2. From the `tutorial-network` directory, run the following command:

    ```
    composer archive create -t dir -n .
    ```
    {:codeblock}

After you run the command, a business network archive file (`tutorial-network@0.0.1.bna`) is created in the `tutorial-network` directory.

## Step Four: Deploying the business network and generating a REST server

Deploy the business network to the {{site.data.keyword.blockchainfull_notm}} Platform by using the [Starter Plan](./develop_starter.html) or [Enterprise Plan](./develop_enterprise.html).

After the business network is deployed to the {{site.data.keyword.blockchainfull_notm}} Platform, a bespoke REST server can be generated. The REST server examines the contents of the business network and builds RESTful API calls for the assets, transactions, and participants of the network. The instructions for generating a REST server are included in the [Hyperledger Composer documentation](https://hyperledger.github.io/composer/latest/integrating/getting-started-rest-api).
