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

# Operating peers on {{site.data.keyword.cloud_notm}} Private with Starter Plan or Enterprise Plan
{: #peer-operate_icp}

***[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

After you set up an {{site.data.keyword.blockchainfull}} Platform on {{site.data.keyword.cloud_notm}} Private (ICP) peer, you need to complete several operational steps before your peer can submit transactions to a Starter Plan or Enterprise Plan network. The steps include adding your organization to a channel, joining your peer to the channel, installing chaincode on your peer, instantiating chaincode on the channel, and connecting applications to your peer.
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

The Fabric SDKs are the recommended path, although the instructions assume familiarity with the operation of the SDK. If you prefer to use the command line, you can use the Fabric peer client.

<!--
It is recommended that you deploy at least two instances of the peer Helm chart for [high availability](peer_icp.html#high-availability). Therefore, you need to follow these operations steps once for each peer. When you are ready to invoke and query chaincode from your application, connect to both peers to ensure that your [applications are highly available](../v10_application.html#ha-app).
-->

**Note**: An {{site.data.keyword.blockchainfull_notm}} Platform peer does not have access to the full functionality or support of peers that are hosted on the {{site.data.keyword.blockchainfull_notm}} Platform. As a result, you cannot use the Network Monitor to operate a peer on ICP. Before you start running peers, ensure that you review the [considerations and limitations](../ibp-for-icp-about.html#ibp-icp-considerations).

## Prerequisites

Whether you plan to use the SDKs or the command line, you need to complete the steps in this section in the course of operating your peer. You can complete all of these steps before you get started.

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
5. Follow the steps configure the [kubeclt CLI](#peer-kubectl-configure) if have not already. You will use kubectl to interact with your peer container.
6. In your CLI, run the first command in the note, which follows **1. Get the application URL by running these commands:** This command will print out the peer URL and port, which is similar to following example. Save this URL for future commands.

  ```
  http://9.30.94.174:30159
  ```
  {:codeblock}

**Note:** If you deploy your peer behind a firewall, you need to open a passthru to enable the network on the platform to complete a TlS handshake with your peer. You only need to enable a passthru for the Node port bound to port 7051 of your peer. For more information, see [finding your peer endpoint information](#peer-endpoint).

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
{: #remote-peer-node-sdk}

Your peer is deployed with the signCert of your peer admin inside. This allows you to use peer admin's certificates and MSP folder to operate the peer.

Locate the certificates you created when you [enrolled your peer admin](peer_deploy_ibp.html#enroll-admin). If you used the example commands, you can find you peer admin MSP folder at `$HOME/fabric-ca-client/peer-admin`.
  - You can build the peer admin user context with the SDK by using the signCert (public key) and private key in the MSP folder. You can find those keys in the following locations:
    - The signCert can be found in the **signcerts** folder: `$HOME/fabric-ca-client/peer-admin/msp/signcerts`
    - The private key can be found in the **keystore:** folder: `$HOME/fabric-ca-client/peer-admin/msp/keystore`
    You can find an example of how to form a user context and operate the SDK by using only the public and private key in [this section of the developing applications tutorial](../v10_application.html#enroll-panel).

You can also use the SDK to generate the peer admin signCert and private key by using the endpoint information of CA on Starter Plan or Enterprise Plan and your [peer admin username and password](peer_deploy_ibp.html#register-admin).

### Uploading the peer admin signCert to Starter Plan or Enterprise Plan
{: #remote-peer-upload-sdk}

You need to upload the peer administrator signCert to the network on {{site.data.keyword.blockchainfull_notm}} Platform so that your application has permission to operate the network.

- You can find your signCert in the directory your SDK generated your crypto material, in the file named after your peer admin. Copy the certificate inside the quotation marks after the `certificate` field, starting with `-----BEGIN CERTIFICATE-----` and ending with `-----END CERTIFICATE-----`. You can use the CLI to convert the certificate into PEM format by issuing the command `echo -e "<CERT>" > admin.pem`. You can then paste the contents of the certificate into your blockchain network by using the Network Monitor. Log in to the network on {{site.data.keyword.blockchainfull_notm}} Platform. On the "Members" screen of the Network Monitor, click **Certificates** > **Add Certificate**. Specify any name for your certificate and paste the contents in to the **Certificate** field. Click **Restart** when your are asked if you want to restart your peers. This action must be performed before your organization joins the channel.

### Passing your peer's TLS cert to the SDK
{: #icp-peer-download-tlscert}

You need to reference the TLS cert of your peers to authenticate communication with your SDK. Locate the TLS cert that you [downloaded from your peer container](#peer-tls) and save it where it can be referenced by your application. If you used the example commands, you can find you peer TLS certificate at `$HOME/fabric-ca-client/peer-tls/peertls.pem`. You can then import the TLS cert into your application by using a simple read file command.

```
var peerTLSCert = fs.readFileSync(path.join(__dirname, './peertls.pem'));
```
{:codeblock}

### Providing the peer endpoint information to the SDK
{: #peer-SDK-endpoints}

Find the [endpoint information of your peer](#peer-endpoint) and provide it to the SDK by declaring a new peer variable or by updating your Connection Profile. The following example defines the peer as an endpoint on your fabric network and passes it the TLS cert that you imported.

```
var peer = fabric_client.newPeer('grpcs://9.30.94.174:30167', { pem:  Buffer.from(peerTLSCert).toString(), 'ssl-target-name-override': null});
```
{:codeblock}

<!--
You need to specify a `ssl-target-name-override` of `<something>.blockchain.com` in order for the peer to resolve the DNS request.
-->

**Note:** Because the peer is outside of {{site.data.keyword.cloud_notm}, other organizations on the {{site.data.keyword.blockchainfull_notm}} Platform network will not be able to find your peer's endpoint information in their Connection Profile. If other organizations need to send your peer a transaction to be endorsed, you need to provide them the peer url in and out of band operation.

### Using the SDK to join to a channel
{: #peer-join-channel-sdk}

Your organization needs to be a member of a channel before you can join the channel with your peer.

  - You can start a new channel for the peer. If you are the channel initiator, you can automatically include your organization during [channel creation](create_channel.html#creating-a-channel).

  - Another member of the blockchain network can also add your organization to an existing channel by using a [channel update](create_channel.html#updating-a-channel).

    After your organization is added to a channel, you need to add your peer's signCert to the channel so that other members can verify your digital signature during transactions. The peer uploads its signCert during installation, which means you only need to synchronize the certificate to the channel. On the "Channels" screen of the Network Monitor, locate the channel that your organization joins and select **Sync Certificate** from the drop-down list under the **Action** header. This action synchronizes the certificates across all the peers on the channel. You might need to wait for a few minutes so that the channel sync can complete before you issue `join channel` commands.

    **Note:** You will only be able to view new blocks being added to the channel, chaincode being instantiated, and other channel updates in the "Channels" screen of your Network Monitor if the peer you are adding to ICP and connecting to a Starter Plan or Enterprise Plan network is part of the same organization as a peer that has been added with the Network Monitor. This is because the Network Monitor gathers information on the "Channels" screen from your peer, and does not have visibility to peers outside of {{site.data.keyword.cloud_notm}}. If none of your peers are using the Private Data feature, the information in the Network Monitor will be the same for one peer in an organization as for the other.

After your organization has become a member of a channel, follow the instructions to [join your peer to a channel](../v10_application.html#join-channel-sdk) by using the SDK. You need to provide the URL of the ordering service and the channel name.

### Using the SDK to install chaincode on the peer
{: #peer-install-cc-sdk}

Use the following instructions to use the SDK to [install a chaincode](../v10_application.html#install-cc-sdk) on your peer.

### Using the SDK to instantiate chaincode on the channel
{: #peer-instantiate-cc-sdk}

Only one member of the channel needs to instantiate or update chaincode. Therefore, any network member of the channel with peers on {{site.data.keyword.blockchainfull_notm}} Platform can use the Network Monitor to instantiate chaincode and specify endorsement policies. However, if you want to use the peer to instantiate chaincode on a channel, you can use the SDK and follow the instructions to [instantiate a chaincode](../v10_application.html#instantiate-cc-sdk).

## Using the CLI to operate the peer
{: #peer-cli-operate}

You can also operate your peer from the command line by using the Fabric `peer` client.

Your peer was deployed with the signCert of your peer admin inside, allowing that identity to operate the peer. The following instructions will use the peer admin MSP folder that was generated when you [deployed your peer](peer_deploy_ibp.html#register-admin) for joining the peer to a channel, install a chaincode on the peer, and then instantiating the chaincode on a channel.

### Downloading the Fabric peer client
{: #peer-client}

The easiest way to to get the peer client is to download all of the Fabric tool binaries from Hyperledger. Navigate to a directory where you would like to download the binaries with your command line and fetch them by issuing the command below. If you haven't installed [Curl ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/prereqs.html#install-curl "Curl"), you'll need to do that first.

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

Downloading the binaries will create a `config` folder that contains a core.yaml, orderer.yaml, and configtx.yaml file. Set the Fabric config path to this `config` directory:

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

Before you can operate the peer, you need to do some management of the certificates on your local machine. For example, you'll need to upload the peer admin signCert to Starter Plan or Enterprise Plan and to ensure that you can access the TLS certificates from the peer and your network on {{site.data.keyword.cloud_notm}}. For more information about the certificates to use and the tasks to perform, see [Managing certificates on {{site.data.keyword.blockchainfull_notm}} Platform](../certificates.html).

1. Move your peer admin's signCert to a new folder that is named `admincerts`:

  ```
  cd $HOME/fabric-ca-client/peer-admin/msp
  mkdir admincerts
  cp signcerts/cert.pem admincerts/cert.pem
  ```
  {:codeblock}

2. Upload this signCert to the network on {{site.data.keyword.cloud_notm}} so that a remote CLI or application can perform channel operations, such as fetching the channel genesis block or instantiating chaincode.
    1. On your local machine, open the file `HOME/fabric-ca-client/peer-admin/msp/admincerts/cert.pem` and copy it to the clipboard.
    2. Enter the Network Monitor of your network on {{site.data.keyword.blockchainfull_notm}} Platform.
    3. Click **Members** in the left navigator and click the **Certificates** tab.
    4. Click the **Add Certificate** button.
    5. In the **Add certificate** window, give your certificate a name, such as "peer-admin", paste in the certificate you just copied to the clipboard and click **Submit**.
    6. You will be asked whether you want to restart the peers. Click **Restart**.
    7. Click **Channels** in the left navigator and locate the channel that the peer will join.
    8. Click the three dots under the **Actions** header and click **Sync Certificate**. In the **Sync certificate** window, click **Submit**.

    **Note**: It is important to sync the channel certificate before the peer joins the channel or instantiates chaincode on the channel. You might need to wait for a few minutes so that channel sync can complete before you issue join channel or instantiate chaincode commands.

3. Ensure that you [downloaded your peer TLS certificate](#peer-tls) and can reference it from your command line. If you followed the example commands, you can find this TLS cert in the `$HOME/fabric-ca-client/peer-tls/peertls.pem` file.

4. You also need to reference the TLS cert you used to communicate with your Starter Plan or Enterprise Plan CA when you [enrolled your peer admin](peer_deploy_ibp.html#enroll-admin). If you followed the example commands in this documentation, you can find the TLS cert in the `$HOME/fabric-ca-client/tls-ibp/tls.pem` file.

You can run a tree command to verify that you have completed these steps. Navigate to the directory where you stored your certificates. A tree command should generate a result similar to the following structure:
```
cd $HOME/fabric-ca-client
tree
.
├── ca-admin
│   ├── fabric-ca-client-config.yaml
│   └── msp
│       ├── cacerts
│       │   └── 9-12-19-115-31873-SampleOrgCA.pem
│       ├── keystore
│       │   └── c44ec1e708f84b6d0359f58ce2c9c8a289919ba81f2cf4bb5187c4ad5a43cbb0_sk
│       └── signcerts
│       |   └── cert.pem
│       └── user
├── catls
│   └── tls.pem
├── peer-admin
│   ├── fabric-ca-client-config.yaml
│   └── msp
│       ├── admincerts
│       │   └── cert.pem
│       ├── cacerts
│       │   └── n4790a319014a473a8a65850c660396ab-org1-ca-us05-blockchain-ibm-com-31011-org1CA.pem
│       ├── keystore
│       │   └── 6c2a999ee80a6b0592c63c58f95193bea2df59efe5489938d513de47b83b372a_sk
│       ├── signcerts
│       │   └── cert.pem
│       └── user
├── peer-tls
│   └── peertls.pem
├── tls-ibp
│   └── tls.pem
├── tls.pem
└── tlsca-admin
    ├── fabric-ca-client-config.yaml
    └── msp
        ├── cacerts
        │   └── 9-30-250-70-30395-tlsca.pem
        ├── keystore
        │   └── bd57fa20283dfc76ada83f989ee0f62ce23e98c94dbd26f6cd23202d8084e38e_sk
        ├── signcerts
        │   └── cert.pem
        └── user
```

### Setting CLI environment variables
{: #environment-variables}

After you move all of our certificates to the necessary location, you need to set some environment variables for your commands to use. Then, you are ready to use the Fabric peer client to operate your peer.

1. Retrieve configuration information from Starter Plan or Enterprise Plan from the **Connection Profile** file that is available in the **Overview** screen of the Network Monitor. Click **Connection Profile** and then **Download**.

  - Find the orderer URL by searching for **orderers**, which is located under `orderers > url`. Make a note of the URL that begins with the network name. The URL is similar to the following example:

    ```
    ash-zbc07b.4.secure.blockchain.ibm.com:21239
    ```

  - Find the name of your organization by searching for **organizations**. This should be the same organization as you use to register your peer. You can find your organization's name together with its associated `mspid`. Make a note of the value of the `mspid`.

2. [Find your peer's endpoint information](#peer-endpoint). You need to use the peer endpoint to set the `PEERADDR` environment variable. Ensure that you exclude the `http://` at the beginning.

3. Run the following commands to set the environment variables.

  ```
  export FABRIC_CFG_PATH=<full_path_to_config_folder>
  export CHANNEL=<CHANNEL_NAME>
  export CC_NAME=<CC_NAME>
  export CORE_PEER_ADDRESS=<PEERADDR>
  export ORDERER_1=<ORDERER_URL>
  export CORE_PEER_MSPCONFIGPATH=<PATH_TO_ADMIN_MSP>
  export CORE_PEER_TLS_ROOTCERT_FILE=<PATH_TO_PEER_TLS_CERT>
  export CORE_PEER_LOCALMSPID=<MSPID_OF_PEER_BEING_USED>
  ```
  {:codeblock}

  Replace the fields with your own information.
    - Replace `<full_path_to_config_folder>` with the config folder that was created when you [downloaded the peer client](#peer-client)
    - Replace `<CHANNEL_NAME>` with the name of the channel that the peer joins.
    - Replace `<CC_NAME>` with any name to refer to your chaincode.
    - Replace `<PEERADDR>` with the peer endpoint from the previous step for the peer you are currently using.
    - Replace `<ORDERER_URL>` with the hostname and port of the orderer from your Connection Profile.
    - Replace `<PATH_TO_ADMIN_MSP>` with the path to your peer admin's MSP folder.
    - Replace `<PATH_TO_PEER_TLS_CERT>` with the path to the peer TLS cert that you downloaded.
    - Replace `<MSPID_OF_PEER_BEING_USED>` with the organization MSPID of the peer you're using to fetch a genesis block or join a channel or install chaincode.

  For example:

  ```
  export FABRIC_CFG_PATH=$HOME/config
  export CHANNEL=defaultchannel
  export CC_NAME=mycc
  export CORE_PEER_ADDRESS=9.30.94.174:30159
  export ORDERER_1=ash-zbc07b.4.secure.blockchain.ibm.com:21239
  export CORE_PEER_MSPCONFIGPATH=$HOME/fabric-ca-client/peer-admin/msp/
  export CORE_PEER_TLS_ROOTCERT_FILE=$HOME/fabric-ca-client/peer-tls/peertls.pem
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

Before you can run the CLI commands to join the peer to a channel, your organization needs to be added to a channel in the network in one of the following ways.

  - You can start a new channel for the peer. As the channel initiator, you can automatically include your organization during [channel creation](create_channel.html#creating-a-channel).
  - Another member of the blockchain network can also add your organization to an existing channel by using a [channel update](create_channel.html#updating-a-channel).

    After your organization is added to a channel, you need to add your peer's signCert to the channel so that other members can verify your digital signature during transactions. The peer uploads its signCert during installation, so that you need to synchronize only the certificate to the channel. On the "Channels" screen of the Network Monitor, locate the channel that your organization joins and select **Sync Certificate** from the drop-down list under the **Action** header. This action synchronizes the certificates across all the peers on the channel.

    **Note:** If your ICP peer that connects to a Starter Plan or Enterprise Plan network is part of the same organization as another peer that is added with the Network Monitor, you can view only new blocks that are added to the channel, chaincode that is instantiated, and other channel updates in the "Channels" screen of your Network Monitor. This is because the Network Monitor gathers information on the "Channels" screen from your peer, and does not have visibility to peers outside of {{site.data.keyword.cloud_notm}}. If none of your peers use the Private Data feature, the information in the Network Monitor is the same for one peer in an organization as for the other peer.

1. Make sure that you have set the [environment variables in your CLI](#environment-variables).

2. Fetch the genesis block of the channel to build the channel ledger on your peer. Note that `$HOME/fabric-ca-client/tls-ibp/tls.pem` represents the path to your Starter Plan or Enterprise Plan TLS certificate. If your path is different, make sure to set the correct one.

  ```
  peer channel fetch 0 -o ${ORDERER_1} -c ${CHANNEL} --cafile $HOME/fabric-ca-client/tls-ibp/tls.pem --tls
  ```
  {:codeblock}

  **Note:** It is possible that when you run any of these CLI commands, you might experience the following warning that can be safely ignored.

  ```
  [msp] getPemMaterialFromDir -> WARN 001 Failed reading file
  /mnt/crypto/peer/peer/msp/intermediatecerts/<intermediate cert name>.pem: no pem content for file  /mnt/crypto/peer/peer/msp/intermediatecerts/<intermediate cert name>.pem
  ```

  Run the following command to verify the genesis block is added to your peer container successfully:

  ```
  ls *.block
  ```
  {:codeblock}

  You know the genesis block is added successfully when you see something that is similar to the following example:

  ```
  defaultchannel_0.block
  ```
  {:codeblock}

3. After you fetch the genesis block of the channel, you can join the peer to the channel by using the following command:

  ```
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

Run the following Peer CLI command to install the `fabcar` chaincode onto the peer:

```
peer chaincode install -n ${CC_NAME} -v v0 -p fabric-samples/chaincode/fabcar/go/
```
{:codeblock}

When this command completes successfully, you should see something similar to:

```
2018-07-06 18:39:00.461 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-07-06 18:39:00.461 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-07-06 18:39:01.142 UTC [main] main -> INFO 003 Exiting.....
```

**Note:** If this chaincode has already been installed and instantiated on another peer running in IBP Starter or Enterprise Plan, there are additional steps that must be performed before installing the chaincode on the peer. See this [troubleshooting topic](#peer-ibp-troubleshooting) for more instructions on how to avoid errors associated with how this chaincode is installed.

### Using the CLI to instantiate chaincode on a channel
{: #icp-toolcontainer-instantiate-cc}

Because only one peer has to instantiate chaincode on a channel, you do not have use your peer to instantiate chaincode. Peers that are hosted on {{site.data.keyword.blockchainfull_notm}} Platform can do that. However, if you would like the peer to instantiate the chaincode onto the channel, you can do so by running the command below. Ensure you have set the value of `CORE_PEER_TLS_ROOTCERT_FILE` to the path of the TLS certificate of your CA on Starter Plan or Enterprise Plan.

```
peer chaincode instantiate -o ${ORDERER_1} -C ${CHANNEL} -n ${CC_NAME} -v v0 -c '{"Args":[""]}' --tls --cafile $HOME/fabric-ca-client/tls-ibp/tls.pem -P ""
```
{:codeblock}

When this command completes successfully, you should see something similar to:

```
2018-07-06 18:43:15.066 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-07-06 18:43:15.066 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
```

After the chaincode has been instantiated, you can use chaincode query and invoke commands to read and write data on the channel ledger. For more information, see the [peer chaincode ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/latest/commands/peerchaincode.html) commands in the Hyperledger Fabric documentation. You will need to pass the orderer endpoint to your invoke commands by using the proxy IP and the external orderer port. You only need to pass the peer endpoint to a query command.

## Updating chaincode

Over time it is likely you need to modify chaincode that is running on the peer. Because chaincode is installed on the peers and instantiated on the channel you need to update the chaincode on all of the peers on the channel.

Complete the following steps to update your chaincode:

1. To update the chaincode on each peer, simply rerun the process you used to install the chaincode on the peers, by using either a client application or a CLI command. Be sure to specify the same chaincode name as was originally used. However, this time increment the chaincode `Version`. All peers need to use the same chaincode name and version.

2. After installing the new chaincode on all the peers in the channel, use the Network Monitor or the
[peer chaincode upgrade ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/peerchaincode.html#peer-chaincode-upgrade) command to update the channel to use the new chaincode.

See step two of these [instructions](install_instantiate_chaincode.html#updating-a-chaincode) for more information about using the "Install Code" panel of Network Monitor to update the chaincode on the channel.

## Viewing the peer logs
{: #peer-ibp-view-logs}

Component logs can be viewed from the command line by using the [`kubectl CLI commands`](#ca-kubectl-configure) or through [Kibana ![External link icon](../images/external_link.svg "External link icon")](https://www.elastic.co/products/kibana "Your window into the Elastic Search"), which is included in your ICP cluster.

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

- Alternatively, you can access logs by using the [ICP cluster management console](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/troubleshoot/events.html), which opens the logs in Kibana.

   **Note:** When you view your logs in Kibana, you might receive the response `No results found`. This condition can occur if ICP uses your worker node IP address as its hostname. To resolve this problem, remove the filter that begins with `node.hostname.keyword` at the top of the panel and the logs will become visible.

## Troubleshooting
{: #peer-ibp-troubleshooting}

### **Problem:** Invoke command fails on peer with a `chaincode fingerprint mismatch` error
{: #icp-cc-install-error}

It is possible to receive a `chaincode fingerprint mismatch` error when you run a `peer chaincode invoke` request on a peer that is running in {{site.data.keyword.cloud_notm}} Private:

```
Error: Error endorsing invoke: rpc error: code = Unknown desc = error executing chaincode: could not get ChaincodeDeploymentSpec for marbles_rp:v0: get ChaincodeDeploymentSpec for marbles_rp/nancyremotepeer from LSCC error: chaincode fingerprint mismatch data mismatch -
```

This error can be caused if there's an inconsistency among the peers that are running the chaincode in one of the following areas:

- Chaincode name
- Chaincode version
- Chaincode path
- Chaincode binary

This error can occur if the Network Monitor UI was used to install and instantiate chaincode on a peer running in Starter or Enterprise plan and then later you install the chaincode on a peer running on {{site.data.keyword.cloud_notm}} Private. The error occurs on the `invoke` request because the resulting chaincode paths on the peers are different.

**Solution:**
If you want to run chaincode on peers in both the {{site.data.keyword.cloud_notm}}, such as Starter or Enterprise Plan, and {{site.data.keyword.cloud_notm}} Private, do not use the Network Monitor UI to install the chaincode. Instead, package chaincode with the [`peer chaincode package`![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/peerchaincode.html?highlight=peer%20chaincode%20package#peer-chaincode-package) command and then install the package on all peers by running the [`peer chaincode install`](#icp-toolcontainer-install-cc) command.

If chaincode is already installed and instantiated on a channel before you attempt to install the chaincode on an {{site.data.keyword.cloud_notm}} Private peer, you need to complete the following steps to avoid the problem:

1. Package chaincode with the [`peer chaincode package`![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/peerchaincode.html?highlight=peer%20chaincode%20package#peer-chaincode-package) command.
2. Install the chaincode package on the peer that are running on {{site.data.keyword.cloud_notm}} Private by running the `peer chaincode install` command.
3. If you have the platform specific binaries, you can run the [`peer chaincode upgrade`![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/peerchaincode.html?highlight=peer%20chaincode%20package#peer-chaincode-upgrade) command to upgrade the chaincode that is running on the Starter or Enterprise plan peer, which uses the chaincode package.
4. Instantiate the newly-installed chaincode on the channel by using either the Network Monitor UI or the CLI.

The process for upgrading chaincode can also be found in [`Chaincode for Operators`![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/chaincode4noah.html) in Hyperledger Fabric documentation.
