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

# Operating peers in {{site.data.keyword.cloud_notm}} Private with a multi-cloud network
{: #peer-operate_icp}

***[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

After you set up an {{site.data.keyword.blockchainfull}} Platform on {{site.data.keyword.cloud_notm}} Private (ICP) peer, you need to complete several operational steps before your peer can issue transactions to query and invoke the ledger of the blockchain network. The steps include adding your organization to a channel, joining your peer to the channel, installing chaincode on your peer, instantiating chaincode on the channel, and connecting applications to your peer. If you want to connect your peer to a Starter Plan or Enterprise Plan network, see [Operating peers on {{site.data.keyword.cloud_notm}} Private with Starter Plan or Enterprise Plan](peer_operate_ibp.html#peer-operate_icp)
{:shortdesc}

You need to complete a few prerequisite steps from your ICP Cluster to operate your peer.

**Prerequisites:**
- [Configure your CLIs](#peer-kubectl-configure)
- [Retrieve your peer endpoint information](#peer-endpoint)
- [Download your peer TLS cert](#peer-tls)

You can then use one of the following methods to operate your peer.

**Operations Paths:**
- [Using the Fabric SDKs](#peer-operate-with-sdk)
- [Using the command line](#peer-cli-operate)

The Fabric SDKs are the recommended path, although the instructions assume familiarity with the operation of the SDK. If you prefer to use the command line, you can use the Fabric peer binaries.

If your organization is not yet a member of a consortium or channel, you can use these steps to [create a channel](#create-channel). The instructions will take you through how to [prepare an organization definition](#organization-definition). This definition will be used to make you a member of the consortium by being added to an orderer system channel. Afterward, you will be able to form a new channel by [creating a channel transaction](#peer-icp-channeltx)
<!--
It is recommended that you deploy at least two instances of the peer Helm chart for [high availability](peer_icp.html#high-availability). Therefore, you need to follow these operations steps once for each peer. When you are ready to invoke and query chaincode from your application, connect to both peers to ensure that your [applications are highly available](../v10_application.html#ha-app).
-->

## Prerequisites

Whether you plan on using the SDKs or the command line, you need to complete the steps in this section in the course of operating your peer. You can complete all of these steps before you get started.

### Configuring the CLIs
{: #peer-kubectl-configure}

You need to use the **kubectl** command line tool to connect to peer container that runs in ICP.

1. Log in to your ICP cluster UI. Navigate to the **Command Line Tools** tab and click **Cloud Private CLI**. You'll see the following tools that you can download.

   * Install IBM Cloud Private CLI and plug-ins
   * Install Kubectl CLI
   * Install Helm
   * Install Istio CLI

  To operate a peer, you need to use the first **three** tools, among which Helm is optional. Click each of them and run the `curl` commands for the machine type that you're using. Then, issue `chmod` and `sudo mv` commands for each tool. The `chmod` command will change the permission of the CLI in question to make it executable, and the `sudo mv` command will move the file and rename it.

  The commands for the first tool **cloudctl** might look like the following example:

  ```
  chmod +x cloudctl-darwin-amd64<suffix of your binary>
  sudo mv path/to/cloudctl-darwin-amd64<suffix> /usr/local/bin/cloudctl
  ```
  {:codeblock}

  The commands for the second tool **kubectl** might look like the following example:

  ```
  chmod +x kubectl-darwin-amd64<suffix of your binary>
  sudo mv path/to/kubectl-darwin-amd64<suffix> /usr/local/bin/kubectl
  ```
  {:codeblock}

2. After you configure **kubectl**,, run the following command:

  ```
  kubectl get pods
  ```
  {:codeblock}

  If successful, the command returns a response that is similar to the following example, where `peer-6cf85f6687-hcxpl` is the name of the peer container.

  ```
  NAME                                 READY      STATUS             RESTARTS   AGE
  peer-6cf85f6687-hcxpl          2/2        Running            0          6h
  ```

  You are now ready to use the **kubectl** tool to get the peer endpoint information.

3. Optionally, if you want to use **Helm**, complete a few more steps. Note that Helm is optional to install and you don't need to use it in these instructions. However, it can be useful to manage your Helm releases and create your own archives to deploy in ICP.

  1. Click "Install Helm" and run the `curl` command from the ICP UI.
  2. Unpack the `tar` file by running the following command:

    ```
    tar -xzvf helm-darwin-amd64<suffix>
    ```
    {:codeblock}

  3. Run the `sudo mv` command by appending `/helm` to `darwin-amd64`. Note that you do not need to run the `chmod` command for Helm. For example:

    ```
    sudo mv darwin-amd64/helm/ /usr/local/bin/helm
    ```
    {:codeblock}

  You can run the `helm help` command to confirm that Helm is installed successfully.

### Retrieving peer endpoint information
{: #peer-endpoint}

You need to target your peer endpoint from the SDK or the Fabric CA client to join channel or install smart contracts. You can find the endpoint of your peer by using your ICP console UI. You will need to be a [Cluster administrator ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/user_management/assign_role.html "Cluster administrator roles and actions") to complete the following steps:

1. Log in to your ICP console and click the **Menu** icon in the upper left corner.
2. Click **Workload** > **Helm Releases**.
3. Find the name of your Helm Release and open the Helm Release details panel.
4. Scroll down to the **Notes** section at the bottom of the panel. In the **Notes** section, you can see a set of commands to help you operate your peer deployment.
5. Follow the steps to configure the [kubeclt CLI](#peer-kubectl-configure) if you have not already. You will use kubectl to interact with your peer container.
6. In your CLI, run the first command in the note, which follows **1. Get the application URL by running these commands:** This command will print out the peer URL and port, which is similar to following example. Save this URL for future commands.

  ```
  http://9.30.94.174:30159
  ```
  {:codeblock}

**Note:** If you deploy your peer behind a firewall, you need to open a passthru to enable the network on the platform to complete a TlS handshake with your peer. You only need to enable a passthru for the Node port bound to port 7051 of your peer. For more information, see [finding your peer endpoint information](peer_operate_ibp.html#peer-endpoint).

### Download your peer TLS cert
{: #peer-tls}

You need to download your peer TLS certificate and pass it to your commands to communicate with your peer from a remote client.

1. Log in to your ICP console and click the **Menu** icon in the upper left corner.
2. Click **Workload** > **Helm Releases**.
3. Find the name of your Helm Release and open the Helm Release details panel.
4. Scroll down to the **Notes** section at the bottom of the panel. In the **Notes** section, you can see a set of commands to help you operate your peer deployment.
5. Follow the steps configure the [kubeclt CLI](#peer-kubectl-configure) if have not already. You will use kubectl to interact with your peer container.
6. In your CLI, run the first command in the note, which follows **3. Get peer TLS root cert file**. This command will save your TLS certificate as the file `cacert.pem` on your local machine.
7. Move the certificate to location where you can reference it in future commands, and rename it `peertls.pem`.

  ```
  mkdir $HOME/fabric-ca-client/peer-tls
  cp cacert.pem $HOME/fabric-ca-client/peer-tls/peertls.pem
  ```
  {:codeblock}

## Using Fabric SDKs to operate the peer
{: #peer-operate-with-sdk}

The Hyperledger Fabric SDKs provide a powerful set of APIs that enable applications to interact with and operate blockchain networks. You can find the latest list of supported languages and the complete list of available APIs within the Fabric SDKs in the [Hyperledger Fabric SDK community documentation ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/getting_started.html#hyperledger-fabric-sdks "Hyperledger Fabric SDK Community documentation"). You can use the Fabric SDKs to join your peer to a channel on the {{site.data.keyword.blockchainfull_notm}} Platform, install a chaincode on your peer, and instantiate the chaincode on a channel.

The following instructions use the [Fabric Node SDK ![External link icon](../images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/ "Fabric Node SDK") to operate the peer and assume prior familiarity with the SDK. You can use the [developing applications tutorial](../v10_application.html) to learn how to use the Node SDK before you get started, and as a guide to developing applications with your peer when you are ready to invoke and query chaincode.

### Installing the Node SDK

You can use NPM to install the [Node SDK ![External link icon](../images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/ "Node SDK"):

```
npm install fabric-client@1.2
```
{:codeblock}

It is recommended that you use version 1.2 of the Node SDK.

### Preparing the SDK to work with the peer
{: #peer-node-sdk}

Your peer was deployed with the signCert of your peer admin inside. This will allow you to use peer admin's certificates and MSP folder to operate the peer.

Locate the certificates that you created when you [enrolled your peer admin](CA_operate.html#enroll-admin). If you used the example commands, you can find you peer admin MSP folder at `$HOME/fabric-ca-client/peer-admin`.
  - You can build the the peer admin user context with the SDK by using the signCert (public key) and private key in the MSP folder. You can find those keys in the following locations:
    - The signCert can be found in the **signcerts** folder: `$HOME/fabric-ca-client/peer-admin/msp/signcerts`
    - the private key can be found in the **keystore:** folder: `$HOME/fabric-ca-client/peer-admin/msp/keystore`
    For an example of how to form a use context and operate the SDK by using only the public and private key, see [Generating certificates by using the Network Monitor](../v10_application.html#enroll-panel).

<!-- You can also use the SDK to generate the peer admin signCert and private key using the endpoint information of CA on Starter Plan or Enterprise Plan and your [peer admin username and password](CA_operate.html#register-admin). -->

### Passing your peer's TLS cert to the SDK
{: #icp-peer-download-tlscert}

You need to reference your peer's TLS cert to authenticate communication from your SDK. Locate the TLS cert that you [downloaded from your peer container](#peer-tls) and save it where it can be referenced by your application. If you used the example commands, you can find you peer TLS certificate at `$HOME/fabric-ca-client/peer-tls/peertls.pem`. You can then import the TLS cert into your application by using a simple read file command.

```
var peerTLSCert = fs.readFileSync(path.join(__dirname, './peertls.pem'));
```
{:codeblock}

### Providing the peer endpoint information to the SDK
{: #peer-SDK-endpoints}

Find the [endpoint information of your peer](#peer-endpoint) and provide it to the SDK by declaring a new peer variable. The following example defines the peer as an endpoint on your fabric network and passes it the TLS cert that you imported.

```
var peer = fabric_client.newPeer('grpcs://9.30.94.174:30159', { pem:  Buffer.from(peerTLSCert).toString(), 'ssl-target-name-override': null});
```
{:codeblock}

<!--
You need to specify a `ssl-target-name-override` of `<something>.blockchain.com` in order for the peer to resolve the DNS request.
-->

If you have a large network, with multiple peers belonging to different organizations that need to endorse your transactions, you may want to work with your consortium to [build a common connection profile ![External link icon](../images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/tutorial-network-config.html "common connection profile")

### Passing your orderer's TLS cert to the SDK
{: #icp-orderer-download-tlscert}

You also need the TLS certificate of the orderer of your consortium to join channels and submit transactions. If you are the administrator of the orderer, follow the instructions to [download the orderer TLS certificate](orderer_operate.html#orderer-tls).  If you used the example commands, you can find you peer TLS certificate at `$HOME/fabric-ca-client/orderer-tls/orderertls.pem`. If the orderer is governed by another organization, they need to provide you the TLS certificate in an out of band operation. You can then import the TLS cert into your application.

```
var ordererTLSCert = fs.readFileSync(path.join(__dirname, './orderertls.pem'));
```
{:codeblock}

### Providing the orderer information to the SDK
{: #orderer-SDK-endpoints}

To use the SDK, you also need the endpoint information of the orderers in your consortium. If you are the administrator of the orderer, you can use these instructions to [Retrieve the orderer endpoint information](orderer_operate.html#orderer-endpoint). If the orderer is governed by another organization, they need to provide you the orderer URL in and out of band operation. The following example defines an orderer as an endpoint and passes it the orderer TLS cert.

```
var orderer = fabric_client.newOrderer('grpcs://9.30.94.174:30167', { pem:  Buffer.from(ordererTLSCert).toString(), 'ssl-target-name-override': null});
```
{:codeblock}
<!--
You need to specify a `ssl-target-name-override` of `<something>.blockchain.com` in order for the peer to resolve the DNS request.
-->

### Using the SDK to join to a channel
{: #peer-join-channel-sdk}

Your organization needs to be a member of a channel before you can join the channel with your peer. If you are not member of a channel, you can follow the instructions to [create a new channel](#create-channel).

After your organization has become a member of a channel, follow the instructions to [join your peers to a channel](../v10_application.html#join-channel-sdk) using the SDK.

### Using the SDK to install chaincode on the peer
{: #peer-install-cc-sdk}

Use the following instructions to use the SDK to [install a chaincode](../v10_application.html#install-cc-sdk) on your peer.

### Using the SDK to instantiate chaincode on the channel
{: #peer-instantiate-cc-sdk}

Only one member of the channel needs to instantiate or update chaincode. Therefore, any member of the channel that has installed the chaincode on their peers can instantiate chaincode and specify endorsement policies. However, if you want to use the peer to instantiate chaincode on a channel, you can use the SDK and follow the instructions to [instantiate a chaincode](../v10_application.html#instantiate-cc-sdk).

## Using the CLI to operate the peer
{: #peer-cli-operate}

You can also operate your peer from the command line by using the Fabric Peer binaries.

Your peer was deployed with the signCert of your peer admin inside, allowing that identity to operate the peer. The following instructions will use the peer admin MSP folder that was generated when you [deployed your peer](CA_operate.html#register-admin) for joining the peer to a channel, install a chaincode on the peer, and then instantiating the chaincode on a channel.

### Downloading the Fabric peer client
{: #peer-client}

To interact with your peer from a remote client, you need to download the [Fabric peer client ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/peercommand.html "Fabric peer command").

The easiest way to get the peer client is to download the Fabric tool binaries from the Hyperledger Project. Create a directory where you would like to download the binaries with your command line, and fetch them by issuing the command below. You need to install [Curl ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/prereqs.html#install-curl "Curl") first.

```
curl -sSL http://bit.ly/2ysbOFE | bash -s 1.2.1 1.2.1 -d -s
```
{:codeblock}

The command will install the binaries in a `bin/` directory. Set your path to your tools, where the above Fabric tools are downloaded:
```
export PATH=$PATH:<full_path_to_bin_folder>
```
{:codeblock}

For example, if you installed the binaries in your home directory you would set your `PATH` as:

```
export PATH=$PATH:$HOME/bin
```
{:codeblock}

Downloading the binaries will create a config folder containing a core.yaml, orderer.yaml and a configtx.yaml file. Set the Fabric config path to this config directory:

```
export FABRIC_CFG_PATH=<full_path_to_config_folder>
```
{:codeblock}

For example:
```
export FABRIC_CFG_PATH=$HOME/config
```
{:codeblock}

### Managing the certificates on your local system
{: #manage-certs}

Switch the directory where the peer admin MSP folder is generated. If you followed example steps in this documentation, you can find the MSP folder in a similar directory as below:

```
cd $HOME/fabric-ca-client/peer-admin/msp
```
{:codeblock}

Before you can operate the peer, you need to do some management of the certificates on your local machine. You also need to ensure that you can access the TLS certificates from the peer. For more information about the certificates to use, see [Membership Service Providers](CA_operate.html#msp) in [Operating a Certificate Authority on {{site.data.keyword.cloud_notm}} Private](CA_operate.html).

1. Move your peer admin's signCert to a new folder that is named `admincerts`:

    ```
    cd $HOME/fabric-ca-client/peer-admin/msp
    mkdir admincerts
    cp signcerts/cert.pem admincerts/cert.pem
    ```
    {:codeblock}

2. Ensure that you [download your peer TLS certificate](#peer-tls) and can reference it from your command line. If you followed the example commands in this documentation, you can find this TLS cert in the `$HOME/fabric-ca-client/peer-tls/peertls.pem` file.

3. You also need to reference to the TLS certificate of your orderer. If you are the administrator of the orderer, follow the instructions to [download the orderer TLS certificate](orderer_operate.html#orderer-tls). If the orderer is governed by another organization, they need to provide you the TLS certificate in an out of band operation. Save this certificate in a location where you can reference it in future commands.

You can run a tree command to verify that you have completed these steps. Navigate to the directory where you stored your certificates. A tree command should generate a result similar to the following structure:
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
├── orderer-tls
│   └── orderertls.pem
├── peer-admin
│   └── msp
│       ├── admincerts
│       │   └── cert.pem
│       ├── cacerts
│       │   └── 9-30-250-70-30395-SampleOrgCA.pem
│       ├── keystore
│       │   └── 24f76217c5c7e641ee29b068712181294e427cbee4dbfce230a1a98b53489cbe_sk
│       ├── signcerts
│       │   └── cert.pem
│       └── user
├── peer-tls
│   └── peertls.pem
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

### Setting CLI environment variables
{: #environment-variables}

After we have moved all of our certificates to the necessary location, we need to set some environment variables that are used by our peer CLI commands. Then we will be ready to use the Fabric peer client to operate our peer.

1. Make sure that you have set the [environment variables in your CLI](#environment-variables)

2. Set the Fabric config path to this config directory that was created when you [downloaded the Fabric tool binaries](#peer-client):

    ```
    export FABRIC_CFG_PATH=<full_path_to_config_folder>
    ```
    {:codeblock}

    Setting this variable will allow you to run the commands using the peer client in any directory.

3. You need the endpoint information of your orderer. If you are the administrator of the orderer, you can use these instructions to [Retrieve the orderer endpoint information](orderer_operate.html#orderer-endpoint). If the orderer is governed by another organization, they need to provide you the orderer URL in and out of band operation.

4. [Retrieve your peers endpoint information](#peer-endpoint). We will use this URL to set the `PEERADDR` environment variable. You need to leave off the `http://` at the beginning.

5. Run the following commands to set the environment variables:

  ```
  export CHANNEL=<CHANNEL_NAME>
  export CC_NAME=<CC_NAME>
  export CORE_PEER_ADDRESS=<PEERADDR>
  export ORDERER_1=<ORDERER_URL>
  export CORE_PEER_MSPCONFIGPATH=<PATH_TO_ADMIN_MSP>
  export CORE_PEER_LOCALMSPID=<MSPID_OF_PEER_BEING_USED>
  ```
  {:codeblock}

  Replace the fields with your own information.
    - Replace `<CHANNEL_NAME>` with the name of the channel that the peer joins.
    - Replace `<CC_NAME>` with any name to refer to your chaincode.
    - Replace `<PEERADDR>` with the peer endpoint hostname and port from the previous step.
    - Replace `<ORDERER_URL>` with the hostname and port of your consortium orderer.
    - Replace `<PATH_TO_ADMIN_MSP>` with the path to your peer admin's MSP folder.
    - Replace `<MSPID_OF_PEER_BEING_USED>` with the organization MSPID of the peer you're using to fetch a genesis block or join a channel or install chaincode. This value was given during the Helm chart deployment.

  For example:

  ```
  export CHANNEL=mychannel
  export CC_NAME=mycc
  export CORE_PEER_ADDRESS=9.30.250.70:32110
  export ORDERER_1=9.30.250.70:31507
  export CORE_PEER_MSPCONFIGPATH=$HOME/fabric-ca-client/peer-admin/msp/
  export CORE_PEER_LOCALMSPID=org1
  ```
  {:codeblock}

  You'll also want to enable TLS, which you can do by issuing:

  ```
  export CORE_PEER_TLS_ENABLED=true
  ```
  {:codeblock}


### Using the CLI to join the peer to the channel
{: #icp-cli-join-peer-to-channel}

Your organization needs to be a member of a channel before you can join the channel with your peer. If you are not member of a channel, you can follow the instructions to [create a new channel](#create-channel).

1. Make sure that you have set the [environment variables in your CLI](#environment-variables).

2. Fetch the genesis block of the channel to build the channel ledger on your peer. Set `CORE_PEER_TLS_ROOTCERT_FILE` to the path of the orderer TLS certificate and run the following command:

  ```
  peer channel fetch 0 -o ${ORDERER_1} -c ${CHANNEL} --tls --cafile $HOME/fabric-ca-client/orderer-tls/orderertls.pem
  ```
  {:codeblock}

  If this is successful, you will see results similar to:
  ```
  2018-11-27 17:00:49.387 EST [cli/common] readBlock -> INFO 041 Received block: 0
  ```

  **Note:** It is possible that when you run any of these CLI commands, you might experience the following warning that can be safely ignored.

  ```
  [msp] getPemMaterialFromDir -> WARN 001 Failed reading file
  /mnt/crypto/peer/peer/msp/intermediatecerts/<intermediate cert name>.pem: no pem content for file  /mnt/crypto/peer/peer/msp/intermediatecerts/<intermediate cert name>.pem
  ```
  {:codeblock}

  Run the following command to verify the genesis block is added to your peer container successfully:

  ```
  ls *.block
  ```
  {:codeblock}

  You know the genesis block is added successfully when you see something that is similar to the following example:

  ```
  defaultchannel_0.block
  ```

3. Once you have fetched the genesis block of the channel, you can join the peer to the channel.  Set `CORE_PEER_TLS_ROOTCERT_FILE` to the path of your peer TLS certificate and join the channel by using the following command:

  ```
  export CORE_PEER_TLS_ROOTCERT_FILE=$HOME/fabric-ca-client/peer-tls/peertls.pem
  peer channel join -b ${CHANNEL}_0.block --tls
  ```
  {:codeblock}

  When this works successfully, you should see information that is similar to the following example:

  ```
  2018-07-06 18:32:09.509 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
  2018-07-06 18:32:09.992 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
  2018-07-06 18:32:09.992 UTC [main] main -> INFO 003 Exiting.....
  ```

### Using the CLI to install chaincode on the peer
{: #icp-toolcontainer-install-cc}

We are now ready to install and instantiate chaincode on the peer. For these instructions, we install the `fabcar` chaincode from the `fabric-samples` repository. Ensure you have [configured your GOPATH ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/dev-setup/devenv.html?highlight=gopath#set-your-gopath "Set your GOPATH") before and then download the `fabric-samples` chaincode from github by using the following commands:

  ```
  cd $GOPATH/src
  git clone https://github.com/hyperledger/fabric-samples
  ```
  {:codeblock}

Run the following Peer CLI command to install the `fabcar` chaincode onto the peer. Set `CORE_PEER_TLS_ROOTCERT_FILE` to the path of your peer TLS certificate.

  ```
  export CORE_PEER_TLS_ROOTCERT_FILE=$HOME/fabric-ca-client/peer-tls/peertls.pem
  peer chaincode install -n ${CC_NAME} -v v0 -p fabric-samples/chaincode/fabcar/go/
  ```
  {:codeblock}

  When this command completes successfully, you should see something similar to:

  ```
  2018-11-27 17:05:54.062 EST [chaincodeCmd] install -> INFO 050 Installed remotely response:<status:200 payload:"OK"
  ```

<!--
**Note:** If this chaincode has already been installed and instantiated on another peer running in IBP Starter or Enterprise Plan, there are additional steps that must be performed before installing the chaincode on the peer. For more instructions about how to avoid errors that are associated with how this chaincode is installed, see the [troubleshooting topic](#icp-cc-install-error).
-->

### Using the CLI to instantiate chaincode on a channel
{: #icp-toolcontainer-instantiate-cc}

Because only one peer has to instantiate chaincode on a channel, you do not have use your peer to instantiate chaincode. Peers that are hosted on {{site.data.keyword.blockchainfull_notm}} Platform can do that. However, if you would like the peer to instantiate the chaincode onto the channel, you can do so by running the command below. Set the value of `CORE_PEER_TLS_ROOTCERT_FILE` to the path of your peer TLS certificate. Set the value of `--cafile` to the orderer TLS cert.

```
cd $HOME/fabric-ca-client
export CORE_PEER_TLS_ROOTCERT_FILE=$HOME/fabric-ca-client/peer-tls/peertls.pem
peer chaincode instantiate -o ${ORDERER_1} -C ${CHANNEL} -n ${CC_NAME} -v v0 -c '{"Args":[""]}' --tls --cafile $PWD/orderer-tls/orderertls.pem -P ""
```
{:codeblock}

When this command completes successfully, you should see something similar to:

```
2018-11-27 17:09:28.013 EST [chaincodeCmd] checkChaincodeCmdParams -> INFO 048 Using default escc
2018-11-27 17:09:28.013 EST [chaincodeCmd] checkChaincodeCmdParams -> INFO 049 Using default vscc
```

After the chaincode has been instantiated, you can use chaincode query and invoke commands to read and write data on the channel ledger. For more information, see the [peer chaincode ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/latest/commands/peerchaincode.html) commands in the Hyperledger Fabric documentation. You will need to pass the orderer endpoint to your invoke commands using the proxy IP and the external orderer port. You only need to pass the peer endpoint to a query command.

## Viewing peer logs in {{site.data.keyword.cloud_notm}} Private
{: #peer-log-icp}

You can access the peer logs from ICP console and check the logs in the Kibana interface.

1. In the ICP console, click the **Menu** icon in the upper left corner.
2. From the menu list, click **Workloads** > **Helm Releases**.
3. In the Helm Releases table, click the name of your helm release.
4. Scroll down to the **Pod** section and click the **View Logs** link at the end of the table row. The peer logs are opened in the Kibana interface.

## Updating chaincode

Over time it is likely you need to modify chaincode that is running on the peer. Because chaincode is installed on the peers and instantiated on the channel you need to update the chaincode on all of the peers on the channel.

Complete the following steps to update your chaincode:

1. To update the chaincode on each peer, simply rerun the process you used to install the chaincode on the peers, by using either a client application or a CLI command. Be sure to specify the same chaincode name as was originally used. However, this time increment the chaincode `Version`. All peers need to use the same chaincede name and version.

2. After you install the new chaincode on all the peers in the channel, use the
[peer chaincode upgrade ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/peerchaincode.html#peer-chaincode-upgrade) command to update the channel to use the new chaincode.

## Restarting a peer that runs in ICP
{: #peer-restart}

Whenever a certificate is added to the peer, the peer node needs to be restarted in order for it to recognize the new certificate.

You can use the **kubectl** commands to restart the peer that runs in ICP. The command requires the name of the peer **pod** and its **container** as command parameters. For more information about using the kubectl commands, see [configuring kubectl CLI](#peer-kubectl-configure).

1. In the ICP console, find the **pod** name and **container** name for your peer deployment.

  1. Click the menu icon in the upper left corner of the console, then click  **Workloads > Deployments**.
  2. Locate and click your peer release in the table to open it.
  3. Scroll down to view the associated **pod** name. Make a note of the **pod** name.
  4. Click the pod to open it.
  5. Click the **Containers** tab. Make a note of your peer **container** name.

2. From your command line, run the following command to restart your peer, replacing `<PEER_POD_NAME>` and `<PEER_CONTAINER_NAME>` with the values from above.

  ```
  kubectl exec <PEER_POD_NAME> -c <PEER_CONTAINER_NAME> /usr/sbin/killall5
  ```
  {:codeblock}

3. You can verify that the peer was restarted by running the `kubectl get pods` command and examining the output for the **RESTART** count for your pod.

## Creating a channel
{: #create-channel}

If your organization is not yet a member of a consortium or channel, you can use the steps below to create a channel. You can also use these steps to update an existing channel. The instructions will take you through how to [prepare an organization definition](#organization-definition). This definition will be used to make you a [member of the consortium](orderer_operate.html#add-organizations-to-consortium) by being added to an orderer system channel. Afterward, you will be able to form a new channel by [creating a channel transaction](#peer-icp-channeltx)

If you have already joined the consortium, you need to complete only the step of preparing the channel transaction. Another member of the consortium can also form a new channel with your organization as a member. You can update the organization definition when you need to send updates to the consortium, such as defining new anchor peers, adding new admin certificates, or updating your crypto material.

### Downloading the configtxgen tool
{: #peer-icp-configtxgen}

If your organization needs to join a consortium or channel, you need to download the [configtxgen ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/configtxgen.html "configtxgen") tool.

The easiest way to get the configtxgen is to download all of the Fabric tool binaries from the Hyperledger Project. Navigate to a directory where you would like to download the binaries with your command line, and fetch them by issuing the command below. You need to install [Curl ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/prereqs.html#install-curl "Curl") first.

```
curl -sSL http://bit.ly/2ysbOFE | bash -s 1.2.1 1.2.1 -d -s
```
{:codeblock}

The command installs the configtxgen in a `bin/` directory. Set the `PATH` path to directory where you downloaded the Fabric tool binaries:

```
export PATH=$PATH:<full_path_to_configtxgen>/bin
```
{:codeblock}

For example, if you installed the binaries in your home directory you would set your `PATH` as:

```
export PATH=$PATH:$HOME/bin
```

## Preparing an organization definition
{: #organization-definition}

Once you have deployed your components, your organization can join a consortium by preparing an organization definition file. The definition contains all the information you need to participate in consortium governance, such as by creating or joining new channels, adding your peers to channels, or instantiating chaincode. The definition contains your organization name, MSPID and MSP certificates. All organizations need to complete this step.

### Prepare your Crypto material

Before you prepare an organization definition, you need to register and enroll the [administrator of your peers](CA_operate.html#register-admin). Navigate to the directory where you created your peer admin MSP folder. The example steps created this folder in `$HOME/fabric-ca-client/peer-admin/msp`. You need to take some additional steps inside this folder before the MSP can be used by the `configtxgen` tool.

1. Copy your TLS cert from your CA and put in a new folder named `tlscacerts`.

  ```
  cd $HOME/fabric-ca-client/peer-admin/msp
  mkdir tlscacerts
  ```

  Then, you'll need to copy the cert to the `tlscacerts` directory you made:

  ```
  cp $HOME/fabric-ca-client/tlsca-admin/cacerts/<xxxx>tlsca.pem tlscacerts/
  ```

  Your command might look something like:

  ```
  cp fabric-ca-client/tlsca-admin/cacerts/9-30-250-70-32129-tlsca.pem /tlscacerts/
  ```
  {:codeblock}

2. Move your signCert from the `signcerts` folder into a new folder named `admincerts`.

  ```
  cd $HOME/fabric-ca-client/peer-admin/msp
  mkdir admincerts
  cp signcerts/cert.pem admincerts/cert.pem
  ```
  {:codeblock}

The `configtxgen` tool creates the definition by consuming a configtx.yaml file. You can find a sample version of this file below:

```
---
################################################################################
#
#   Section: Organizations
#
#   - This section defines the different organizational identities which will
#   be referenced later in the configuration.
#
################################################################################
Organizations:
    - &org1
        # DefaultOrg defines the organization which is used in the sampleconfig
        # of the fabric.git development environment
        Name: org1

        # ID to load the MSP definition as
        ID: org1

        MSPDir: <path_to_peer-admin>/msp

        AnchorPeers:
            # AnchorPeers defines the location of peers which can be used
            # for cross org gossip communication.  Note, this value is only
            # encoded in the genesis block in the Application section context
            - Host: 9.30.250.70
              Port: 30481
```
{:codeblock}

This file contains the information that define your organization within the consortium. A more complex version of this file is also available in the `/config` folder of [the fabric peer client that you downloaded](peer_operate_icp.html#peer-client). You can choose to edit that file, or replace it with the sample above. Note the location of this `/config` folder to set the value of the `FABRIC_CFG_PATH` below. Edit the `Organizations` section of this file and set the following values:

- The `Name:` can be whatever name you wish to use for your organization.

- The `ID:` values of your MSPID, which become the unique identifier of your organization. This Organization MSP value was specified when you deployed your peer helm chart. It is also visible inside the peer container by running the following commands, replacing `<peer pod name>` with the value of your peer's pod:

  ```
  kubectl exec -it <peer pod name> -c peer sh
  $ env | grep CORE_PEER_LOCALMSPID
  ```
  {:codeblock}

  The output might look similar to:
  ```
  CORE_PEER_LOCALMSPID=org1
  ```

  Therefore, the MSP `ID` is org1.

- The `MSPDir:` is the fully qualified path to your admin MSP, for example: `/Users/chandra/fabric-ca-client/peer-admin/msp`

- The `AnchorPeers:` field is the peer that your organization selects to be the lead peer for inter organization communication by gossip. This is used for features such as private data or service discovery; however, private data and service discovery are not currently supported by the peer Helm chart, although the field is still required. The `Host` and `Port` values are available in the ICP peer Helm Release page under the Notes section `#1. Get the application URL by running these commands`. The output will be similar to:

```
http://9.30.250.70:30481
```

Where the `Host:` = `9.30.250.70` and `Port:` = `30481`

Then use the following command to print the organization definition:

```
export FABRIC_CFG_PATH=<path_to_config_folder>
configtxgen -printOrg <org_name> > <path_to_org_definition>orgdefinition.json
```
{:codeblock}

Your call might look similar to the following example command:

```
export FABRIC_CFG_PATH==<full_path_to_config_folder>
mkdir -p $HOME/fabric-ca-client/org-definitions
configtxgen -printOrg org1 > $HOME/fabric-ca-client/org-definitions/org1definition.json
```
{:codeblock}

If this command is successful, the `configtxgen` will print the organization definition in JSON format. You need to send this file to the orderer organization in an out of band operation to join the consortium. The orderer organization can then [form a consortium or be added to an existing consortium](orderer_operate.html#consortium) by adding the definition to the system channel, allowing you to create new channels and be added to channels by other consortium members.

## Creating the channel transaction
{: #peer-icp-channeltx}

Before you can create a new channel, your organization should have prepared an [organization definition](#organization-definition) and become a member of the consortium. Follow these instructions if you need [form a consortium or be added to one](orderer_operate.html#consortium). Members of the consortium can also be easily added to new channels, if their organization has already been added to the system channel. Organizations that are not members of the system channel can only join a channel manually, by adding their organization definition to the channel using a [channel update request ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/channel_update_tutorial.html). You can also use these steps to update an existing channel.

### Forming a new channel
{: #peer-icp-create-channel}

To form a new channel, an organization needs to create a channel creation transaction proposal using the [configtxgen tool![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/configtxgen.html "configtxgen"). This tool consumes a `configtx.yaml` file that defines the new channel members. A sample `configtx.yaml` file is provided below. A more complex version of this file is also available in the `/config` folder of [the fabric peer client that you downloaded](peer_operate_icp.html#peer-client). You can choose to edit that file, or replace it with the sample. Note the location of this `/config` folder to set the value of the `FABRIC_CFG_PATH` below.
```
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#
---
################################################################################
#
#   Section: Organizations
#
#   - This section defines the different organizational identities which will
#   be referenced later in the configuration.
#
################################################################################
Organizations:
    - &org1

        Name: org1

        # ID to load the MSP definition as
        ID: org1

        MSPDir: /Users/chandra/fabric-ca-client/peer-admin/msp

        AnchorPeers:

            - Host: 9.30.250.70
              Port: 32110


################################################################################
#
#   SECTION: Capabilities
#
################################################################################
Capabilities:
    # Channel capabilities apply to both the orderers and the peers and must be
    # supported by both.  Set the value of the capability to true to require it.
    Global: &ChannelCapabilities

        V1_1: true

    # Orderer capabilities apply only to the orderers, and may be safely
    # manipulated without concern for upgrading peers.  Set the value of the
    # capability to true to require it.
    Orderer: &OrdererCapabilities

        V1_1: true

    # Application capabilities apply only to the peer network, and may be safely
    # manipulated without concern for upgrading orderers.  Set the value of the
    # capability to true to require it.
    Application: &ApplicationCapabilities

        V1_2: true

################################################################################
#
#   SECTION: Application
#
#   - This section defines the values to encode into a config transaction or
#   genesis block for application related parameters
#
################################################################################

Application: &ApplicationDefaults

    # Organizations is the list of orgs which are defined as participants on
    # the application side of the network
    Organizations:

    Capabilities:
        <<: *ApplicationCapabilities


################################################################################
#
#   Profile
#
#   - Different configuration profiles may be encoded here to be specified
#   as parameters to the configtxgen tool
#
################################################################################

Profiles:

    mychannel:
        Consortium: SampleConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *org1
            Capabilities:
                <<: *ApplicationCapabilities

```
{:codeblock}

The three sections that are relevant for creating a new channel are **Organizations**, **Capabilities**, and **Profiles**:

- **Organizations:** This section defines all of the members of the consortium. Each organization has an anchor, such as `&Org1`. Under this anchor you can find the organization's name, MSPID, directory to their MSP folder, and the anchor peers of their organization for cross peer gossip. You can fill out the organization profile for each consortium member with the following steps:
  1. Specify the `Name` and `ID` of the organization by using their MSPID. The organization that creates the channel needs to know the MSPID that was specified when the peer Helm chart was deployed.
  2. In `MSPDir`, specify the fully qualified path to the MSP folder you used to create your [organization definition](#organization-definition). Note that none of your crypto material is used in the channel creation transaction. The *configtxgen* tool will ignore the actual contents of the MSP. However it expects an MSP folder at that location, with the correct folder substructure.
  3. The `AnchorPeers` parameter is used to identity the lead peer each organization uses for inter organization communication by gossip. Specify the hostname and port of the peer that will act as the [anchor peer ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/glossary.html) for this organization. This value is important for using features such as private data or service discovery; however, private data and service discovery are not currently supported by the peer Helm chart.

- **Capabilities:** This section is required to be in the `configtx.yaml` but no changes are required.

- **Profiles:** section. This section contains the information that is needed to create a new channel. The profile contains the following information:
  1. The name of the new channel.
  2. The name of the consortium that is defined within the system channel and will be used to create the channel.
  3. The list of organizations that will be added to the channel. The listed organization names will use the anchor peers that are listed in the `Organizations` section to find the listed organizations MSPID and the associated anchor peers.

  The *configtxgen* tool uses this `Profiles` section to create the channel creation proposal.

After updated the configuration file, generate the channel update transaction proposal using the following command:

``` 
export FABRIC_CFG_PATH=<path_to_config_folder>
configtxgen -profile <channel_profile_name> -outputCreateChannelTx ./<channel_profile_name>.tx -channelID <channel_profile_name>
```
{:codeblock}

A real command would like similar the to following example:
``` 
export FABRIC_CFG_PATH=$HOME/config
configtxgen -profile mychannel -outputCreateChannelTx ./mychannel.tx -channelID mychannel
```
{:codeblock}


If successful, you will see a result similar to the following output below. The command will write the channel update transaction proposal in your `FABRIC_CFG_PATH` directory.

```
2018-11-27 16:39:36.130 EST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-11-27 16:39:36.134 EST [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
2018-11-27 16:39:36.134 EST [common/tools/configtxgen/encoder] NewApplicationGroup -> WARN 003 Default policy emission is deprecated, please include policy specificiations for the application group in configtx.yaml
2018-11-27 16:39:36.135 EST [common/tools/configtxgen/encoder] NewApplicationOrgGroup -> WARN 004 Default policy emission is deprecated, please include policy specificiations for the application org group org1 in configtx.yaml
2018-11-27 16:39:36.136 EST [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 005 Writing new channel tx
```

If your endorsement policy does not require additional signatures from other members of the consortium, you can submit the proposal directly to the orderer to form the new channel. Run the following command from the same location as you generated the channel update transaction to create the channel.

```
export CORE_PEER_LOCALMSPID=<ORG_MSPID>
export CORE_PEER_MSPCONFIGPATH=<PATH_TO_ADMIN_MSP>
peer channel create -o <orderer_url> --tls --cafile <orderer_tls_cert> -c <channel_profile_name> -f ./<channel_profile_name>.tx
```
{:codeblock}

Where `<ORG_MSPID>` is the MSPID of your organization, and `<PATH_TO_ADMIN_MSP>` is the path to your peer admin MSP folder. Inside the command, `<orderer_url>` is the URL of the orderer that will create the channel, and `<orderer_tls_cert>` is the path the orderers TLS certificate. A real command might look like the example below:
```
export CORE_PEER_LOCALMSPID=org1
export CORE_PEER_MSPCONFIGPATH=$HOME/fabric-ca-client/peer-admin/msp/
peer channel create -o 9.30.250.70:31507 --tls --cafile $PWD/orderer-tls/orderertls.pem -c mychannel -f ./mychannel.tx
```
{:codeblock}

```
2018-11-27 16:59:30.610 EST [cli/common] readBlock -> INFO 04f Received block: 0
```


## Endorsing the proposal and submitting the request

When the new channel creation proposal is signed by enough organizations to meet the system channel endorsement policy, you can submit the channel update transaction to the orderer to form the new channel

You can use the Node Fabric SDK APIs to complete signing and submission in a single transaction. For more information, visit the [How to create a Hyperledger Fabric Channel ![External link icon](../images/external_link.svg "External link icon")]](https://fabric-sdk-node.github.io/release-1.3/tutorial-channel-create.html) tutorial in the Node SDK documentation.

## Viewing the peer logs
{: #peer-icp-view-logs}

Component logs can be viewed from the command line by using the [`kubectl CLI commands`](#peer-kubectl-configure) or through [Kibana ![External link icon](../images/external_link.svg "External link icon")](https://www.elastic.co/products/kibana "Your window into the Elastic Search"), which is included in your ICP cluster.

- Use the `kubectl logs` command to view the container logs inside the pod. If you are unsure of your pod name, run the following command to view your list of pods.

  ```
  kubectl get pods
  ```
  {:codeblock}

  Then, run the following command to retrieve the logs for the peer container that resides inside the pod by replacing `<pod_name>` with the name of your pod from the command output above:

  ```
  kubectl logs <pod_name> -c peer
  ```
  {:codeblock}

  For more information about the `kubectl logs` command, see [Kubernetes documentation ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs “Getting Started”)

- Alternatively, you can access logs by using the  [ICP cluster management console](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/troubleshoot/events.html), which opens the logs in Kibana.

  **Note:** When you view your logs in Kibana, you might receive the response `No results found`. This condition can occur if ICP uses your worker node IP address as its hostname. To resolve this problem, remove the filter that begins with `node.hostname.keyword` at the top of the panel and the logs will become visible.


## Troubleshooting
{: #peer-icp-troubleshooting}

<!-- Commenting out per issue #228
### **Problem:** Error when installing helm release by using the helm install command
{: #icp-invalid-helm}

Using the `helm install` command to install the helm release fails with the error:

```
Error: incompatible versions client[v2.9.1] server[v2.7.3+icp]
```

**Solution:**

This error can occur when the helm CLI version on your local machine is
ahead of helm cli version on your server. To resolve the issue, downgrade the
version of your helm CLI by performing the following steps:

1.  Remove current version of helm client from `/usr/local/bin`.  Or you can
optionally just move binary file to a different directory.
```
rm /usr/local/bin/helm
```
{:codeblock}
2.  Follow these instructions to install helm client by using a script that is provided
by [helm](https://docs.helm.sh/using_helm/#from-script), which includes the following commands:

   * `curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh`
   * `chmod 700 get_helm.sh`
   * `./get_helm.sh --version v2.7.2`

   In the last step, ensure the selected version is compatible with ICP 2.1.0.3. `v2.7.2 ` is recommended.

3.  After the helm installation completes, [login](#peer-kubectl-configure}) to your ICP cluster
and run the following command:

 ```
 bx pr cluster-config <cluster_name>
 ```
 {:codeblock}
 sample output:

 ```
 root@xcephu27:~/helmcli# bx pr cluster-config condev2
 Configuring kubectl: /root/.bluemix/plugins/icp/clusters/condev2/kube-config
 Property "clusters.condev2" unset.
 Property "users.condev2-user" unset.
 Property "contexts.condev2-context" unset.
 Cluster "condev2" set.
 User "condev2-user" set.
 Context "condev2-context" created.
 Switched to context "condev2-context".

 Cluster condev2 configured successfully.

 Configuring helm: /root/.helm
 Helm configured successfully

 OK
 ```

4. Add helm home directory from step three to your env:

 ```
 export HELM_HOME=/root/.helm
 ```
 {:codeblock}

5. Check helm version:

  ```
  helm version --tls
  ```
  {:codeblock}

  sample output:

  ```
  root@xcephu27:~/helmcli# helm version --tls
  Client: &version.Version{SemVer:"v2.7.2", GitCommit:"8478fb4fc723885b155c924d1c8c410b7a9444e6", GitTreeState:"clean"}
  Server: &version.Version{SemVer:"v2.7.3+icp", GitCommit:"27442e4cfd324d8f82f935fe0b7b492994d4c289", GitTreeState:"dirty"}
  ```

6. Rerun your `helm install` command.

-->

### **Problem:** Error when logging in to peer container
{: #icp-bash-error}

This problem occurs when you use the kubectl tool to exec into your peer by running the following command:

```
kubectl exec -it -n <namespace> <PEER_CONTAINER_NAME> bash
```
{:codeblock}

Fails and produces an error message similar to the following example:

```
OCI runtime exec failed: exec failed: container_linux.go:348: starting container process caused "open /dev/pts/4294967296: no such file or directory": unknown
```

**Solution:**

This error occurs when on big endian systems, such as IBM System Z, when two Docker bash sessions are opened in the same Docker container. The second bash session might be unable to connect to the running container. To resolve this issue, [restart the peer container](#peer-restart) and retry the `kubectl exec` command after the peer restarts.
