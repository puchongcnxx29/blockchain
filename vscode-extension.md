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

# Tools for smart contracts
{: #overview}


***[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


The {{site.data.keyword.blockchainfull}} Platform Visual Studio Code extension provides an environment within Visual Studio Code for developing, packaging, and deploying smart contract packages. The extension also includes commands to set up a preconfigured local instance of Hyperledger Fabric for simplified local smart contract development.

**Please note:** The {{site.data.keyword.blockchainfull_notm}} Platform extension is compatible with {{site.data.keyword.blockchainfull_notm}} Platform Starter Plan networks, and with Hyperledger Fabric 1.3.x.

## Prerequisites

Before installing the {{site.data.keyword.blockchainfull_notm}} Platform Visual Studio Code extension, complete the following prerequisites.

- **Install Yeoman**

  Yeoman is a generator tool used to create skeleton smart contract projects. Install Yeoman using the following command: `npm install -g yo`

- **Install the Fabric generator for Yeoman**

  After installing Yeoman, install the Fabric generator for creating skeleton smart contract packages. Install the Fabric generator using the following command: `npm install -g generator-fabric`

- **Install Docker**

  To run the preconfigured instance of Hyperledger Fabric, ensure you have installed [Docker](https://www.docker.com/).

- **Operating system requirements**

  Currently, this extension is compatible with Mac, Windows, and Linux.

- **Hyperledger Fabric version requirements**

  This extension is compatible with Hyperledger Fabric versions 1.3.0 and later.

- **Hyperledger Fabric version requirements**

  This extension is compatible with Hyperledger Fabric versions 1.3.0 and later.


## Installing the extension

1. Navigate to the [Visual Studio Code extension marketplace page](https://marketplace.visualstudio.com/items?itemName=IBMBlockchain.ibm-blockchain-platform) or search for **{{site.data.keyword.blockchainfull_notm}} Platform** in the extensions panel within Visual Studio Code.
2. Click **Install**.
3. Restart Visual Studio Code to complete installation of the extension.

## Visual Studio Code command palette

This extensions adds many commands to the Visual Studio Code command palette. Many of the operations detailed in this document can also be started by running the command from the Visual Studio Code command palette.

## Creating a project

To create a new project, complete the following steps:

1. Open the command palette, run the **Blockchain: Create Smart Contract Project** command.
2. Select the language you wish to create a smart contract in, currently the options are JavaScript or TypeScript.
3. Create a folder with the name of your project, and open it.
4. Select how to open your new project. The project folder should now open.

## Connecting to and disconnecting from a network

When you use this extension, you install and instantiate smart contract packages on peers and channels in your Hyperledger Fabric instance. This extension can initialise a preconfigured local instance of Hyperledger Fabric using Docker.

### Connecting to the preconfigured Hyperledger Fabric instance

To connect to the preconfigured Hyperledger Fabric instance, first ensure Docker is running on your machine:

1. Open the _{{site.data.keyword.blockchainfull_notm}} Platform_ tab in Visual Studio Code.
2. In the _{{site.data.keyword.blockchainfull_notm}} Platform_ pane, click **local_fabric**. If Docker is running, the local Hyperledger Fabric instance should be downloaded and started.
3. After the local Hyperledger Fabric instance has been started, double click **local_fabric** to connect to it. You should now see a channel called `mychannel`.

#### Restarting the preconfigured Hyperledger Fabric runtime

To restart the `local_fabric` runtime:

1. Right-click on the `local_fabric` connection when it is established.
2. Select **Restart Fabric Runtime**.

The Hyperledger Fabric containers will be stopped and restarted.

#### Teardown the Hyperledger Fabric runtime

To teardown the `local_fabric` runtime:

1. Right-click on the `local_fabric` connection when it is established.
2. Select **Teardown Fabric Runtime**.

Performing a teardown of the `local_fabric` network will close all Hyperledger Fabric containers. **Please note**: This will result in the loss of the ledger and world state data.

#### Enabling development mode on the preconfigured Hyperledger Fabric runtime

In normal operation, a peer will create and maintain a chaincode container to run instantiated smart contracts. By switching to development mode, the peer allows the chaincode container to be run manually. Running the chaincode container manually, on the command line or in a terminal, assists with iterative development and debugging of smart contracts.

To enable development mode with the preconfigured Hyperledger Fabric runtime:

1. Right-click on the `local_fabric` connection when it is established.
2. Select **Toggle Development Mode**.

Development mode must be enabled in order to use the debugging feature of the extension.

### Connecting to {{site.data.keyword.blockchainfull_notm}} Platform Starter Plan

The {{site.data.keyword.blockchainfull_notm}} Platform Visual Studio Code extension can be used to connect to an instance of the {{site.data.keyword.blockchainfull_notm}} Platform Starter Plan. After connected, the extension can be used to develop and deploy smart contracts.

**Note:** Currently the extension is not compatible with {{site.data.keyword.blockchainfull_notm}} Platform Enterprise Plan.

1. If you don't have an instance of {{site.data.keyword.blockchainfull_notm}} Platform Starter Plan, create one [here](https://console.bluemix.net/catalog/services/blockchain).
2. Open the IBP UI by clicking **Launch**.
3. Retrieve your connection profile by clicking **Overview** > **Connection Profile** > **Download**.
4. Generate the required admin certificates by clicking **Certificate Authority** > **Generate Certificate** and saving the certificate and private key to your file system.
5. Add the certificates to your {{site.data.keyword.blockchainfull_notm}} Platform Starter Plan instance by clicking **Members** > **Certificates** > **Add Certificate**, and selecting the certificate created in the preceding step.
6. In Visual Studio Code, open the {{site.data.keyword.blockchainfull_notm}} Platform extension view and click **Add new connection**.
7. Enter the connection name, and the path to the connection profile, certificate, and private key.

### Connecting to your own Hyperledger Fabric instance

By using this extension, you can connect to a preconfigured local instance of Hyperledger Fabric, or you can connect to your own Hyperledger Fabric instance. If you choose to connect to your own Fabric instance, it must be Fabric 1.3.0 or later.

To connect to your own Hyperledger Fabric instance, complete the following steps:

1. Open the _{{site.data.keyword.blockchainfull_notm}} Platform_ tab in Visual Studio Code.
2. In the _{{site.data.keyword.blockchainfull_notm}} Platform_ pane, click **Add new connection**.
3. Enter a name for the connection. This name will be displayed in the _{{site.data.keyword.blockchainfull_notm}} Platform_ pane.
4. Enter the fully qualified filepath of your Hyperledger Fabric connection profile.
5. Enter the fully qualified filepath of the certificate for your identity.
6. Enter the fully qualified filepath of the private key for use with the certificate supplied in the preceding step.
7. Your connection should not appear in the connections list underneath `local_fabric`. Double-click the connection name to connect.

If you don't have a preexisting Hyperledger Fabric instance of version 1.3.0 or later, you can connect using the preconfigured Hyperledger Fabric instance, or run the following commands to pull and tag the version 1.3.0 images:

```
docker pull nexus3.hyperledger.org:10001/hyperledger/fabric-ca:amd64-1.3.0-stable
docker tag nexus3.hyperledger.org:10001/hyperledger/fabric-ca:amd64-1.3.0-stable hyperledger/fabric-ca
docker pull nexus3.hyperledger.org:10001/hyperledger/fabric-orderer:amd64-1.3.0-stable
docker tag nexus3.hyperledger.org:10001/hyperledger/fabric-orderer:amd64-1.3.0-stable hyperledger/fabric-orderer
docker pull nexus3.hyperledger.org:10001/hyperledger/fabric-peer:amd64-1.3.0-stable
docker tag nexus3.hyperledger.org:10001/hyperledger/fabric-peer:amd64-1.3.0-stable hyperledger/fabric-peer
docker pull nexus3.hyperledger.org:10001/hyperledger/fabric-tools:amd64-1.3.0-stable
docker tag nexus3.hyperledger.org:10001/hyperledger/fabric-tools:amd64-1.3.0-stable hyperledger/fabric-tools
docker pull nexus3.hyperledger.org:10001/hyperledger/fabric-ccenv:amd64-1.3.0-stable
docker tag nexus3.hyperledger.org:10001/hyperledger/fabric-ccenv:amd64-1.3.0-stable hyperledger/fabric-ccenv
```
{:codeblock}

### Disconnecting from a network

When connected to a network, you can close the connection by clicking the disconnect icon in the upper-right of the Blockchain Connections pane. You can also disconnect by running the **Disconnect from a blockchain** command from the command palette.

## Editing or deleting a connection

Connections can be edited or deleted in the connections pane.

### Editing a connection

By editing a connection, you can change the filepath for the connection profile, the name given to the connection, and the filepaths to identity certificates and private keys.

To edit a connection:

1. From the extension, right-click on the connection to edit in the lower-left, this opens a contextual menu with the options to add an identity, edit the connection, or delete the connection.
2. Select **Edit connection**.
3. The _User Settings_ page will open, with the connection details highlighted.
4. Make any changes, and save the settings page.

### Deleting a connection

Connections can be deleted by:

1. From the extension, right-click on the connection to edit in the lower-left, this opens a contextual menu with the options to add an identity, edit the connection, or delete the connection.
2. Select **Delete connection**.
3. A dialogue box appears to confirm connection deletion. Click **Yes**.

The connection will be deleted.

## Adding identities

When you add a connection, you must submit a connection profile, certificate, and private key. The identity that is associated with the certificate/private key pair will be set as the default identity to use when connecting to that Hyperledger Fabric instance.

To add an identity to a connection that is already established:

1. From the extension, right-click on the connection to edit in the lower-left, this opens a contextual menu with the options to add an identity, edit the connection, or delete the connection.
2. Select **Add an identity**.
3. Provide the filepaths for the identity certificates and private key.

## Exploring connections

After connecting to a Hyperledger Fabric instance, you can see a list of the available channels and peers in those channels in the _{{site.data.keyword.blockchainfull_notm}} Platform_ pane. Spend a minute exploring your Hyperledger Fabric instance to get familiar with the structure. Listed first are the channels within a connection, under a channel is a list of the peers that are members of that channel, and a list of smart contract packages instantiated on that channel. Under each peer in the list is a list of all smart contract packages installed on that peer. You can also see the versions of smart contract packages that are installed or instantiated.

### Exporting connection details for the preconfigured Hyperledger Fabric runtime

The connection details required to connect to the `local_fabric` connection can be exported. Connection details for the `local_fabric` connection are useful for testing client applications intended to connect to or interact with your Hyperledger Fabric instance.

To export the `local_fabric` connection details:

1. Start the preconfigured Hyperledger Fabric runtime.
2. Right-click on the `local_fabric` connection, and select **Export Connection Details**.

The connection details are saved to a directory called `local_fabric` contained in your current project directory.

## Packaging a smart contract

When a smart contract is ready to be deployed, it must first be packaged. To package a smart contract, complete the following steps:

1. In Visual Studio Code, navigate to the _{{site.data.keyword.blockchainfull_notm}} Platform_ panel.
2. In the _Smart Contract Packages_ pane, click the + icon. If you have a smart contract project open in the file viewer, it will be automatically packaged, and will be shown in the _Smart Contract Packages_ pane. If you have multiple smart contract folders open, you will be asked which one to package. If you have no smart contract folders open, you'll get an error message.

## Installing smart contract packages

After connecting to an instance of Hyperledger Fabric, you can install and instantiate smart contract packages on the peers.

1. In the _{{site.data.keyword.blockchainfull_notm}} Platform_ pane, navigate to the peer you wish to install the smart contract package on.
2. Right-click the peer to install the smart contract package on, and select **Install Smart Contract**.

## Instantiating smart contract packages

To begin running on a channel, an installed smart contract package must first be instantiated.

1. Connect to the Hyperledger Fabric instance where the smart contract package is installed.
2. Right-click on the channel you wish to instantiate the smart contract package on and click **Instantiate Smart Contract**.
3. Select the smart contract package and version to be instantiated. The smart contract package must be installed on a peer that is a member of this channel.
4. Enter the name of the instantiate function in your smart contract.
5. Enter any arguments required by your instantiate function.

## Exporting or deleting a smart contract package

After a smart contract has been packaged, it can be exported as a `.cds` file or deleted if it is no longer needed.

To delete a smart contract package:

1. In the {{site.data.keyword.blockchainfull_notm}} Platform extension panel, right-click the smart contract package to delete.
2. Select **Delete Package**.

The package will now be deleted and disappear from the list of smart contract packages.

To export a smart contract package:

1. In the {{site.data.keyword.blockchainfull_notm}} Platform extension panel, right-click the smart contract package to export.
2. Select **Export Package**.
3. Choose the directory to save your smart contract package file and click **Export**.

## Testing an instantiated smart contract

Tests for a smart contract can be generated after the smart contract has been instantiated. The tests can be generated as either JavaScript or TypeScript, and run or debugged.

To generate smart contract tests:

1. Ensure that the smart contract has been instantiated.
2. Under **Instantiated Smart Contracts**, right-click on the smart contract to generate tests for.
3. Select **Generate Smart Contract Tests**.
4. Now select the language for the test file, either **JavaScript** or **TypeScript**. The {{site.data.keyword.blockchainfull_notm}} Platform extension will now install required npm modules and build the test file.

After the test file has been built, the tests can be run by clicking the **Run Tests** button in the file.

## Debugging a smart contract using the preconfigured Hyperledger Fabric runtime

Debugging a smart contract allows you to run through the smart contract transactions with breakpoints and output, ensuring the transactions work as intended. To debug your smart contract:

1. Ensure you are connected to the `local_fabric` connection that it is in development mode.
2. Open your smart contract project.
3. Open the debug view in Visual Studio Code using the left-hand navigation bar.
4. Select the Debug Smart Contract configuration by using the dropdown in the upper-left.
5. Package and install the smart contract by clicking the **play** button.
6. Add breakpoints to the smart contract by clicking on the relevant line numbers in your smart contract files.
7. Right-click on the installed smart contract and select **Instantiate**. You can now right-click to submit transactions and the execution will be paused on defined breakpoints.

To make changes to your smart contract while debugging, click the **restart** button after making the changes to your smart contract. Restarting debugging means you don't need to instantiate the contract again.

**Please note**: Restarting debugging stores the smart contract in local memory, for a large number of changes to large smart contracts, you may need to reinstantiate the smart contract.

## Upgrading an instantiated smart contract

After a smart contract has been installed on a peer and instantiated on a channel, it can be upgraded to deploy a newer version of the smart contract.

1. Ensure that the smart contract you wish to upgrade is instantiated.
2. Install the new version of the smart contract to a peer on the same network.
3. Right-click on the instantiated smart contract, and select **Upgrade Smart Contract**.
4. Optionally, choose a transaction to run as the new smart contract is instantiated.

## Submitting transactions

After a smart contract has been installed and instantiated, the transactions can be submitted from the connections pane in the {{site.data.keyword.blockchainfull_notm}} Platform extension panel.

To submit a transaction:

1. Ensure that your smart contract is installed and instantiated, and that you're connected to the network.
2. In the connections pane, expand the **Instantiated Smart Contracts**.
3. Expand the smart contract that contains the transaction to be submitted.
4. Right-click on the transaction to submit, and select **Submit Transaction**.
5. Enter any arguments required by the transaction, and press **Enter**.
