---

copyright:
  years: 2017, 2018
lastupdated: "2018-11-27"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# Operating peers in AWS
{: #remote-peer-operate-aws}


***[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


After you set up {{site.data.keyword.blockchainfull}} Platform peers in AWS, you need to complete several operational steps before your peer can issue transactions to query and invoke the ledger of the blockchain network. The steps include adding your organization to a channel, joining your peer to the channel, installing chaincode on your peer, instantiating chaincode on the channel, and connecting applications to your peer. You can use either the [Fabric SDKs](#aws-peer-operate-with-sdk) or the [command line](#aws-peer-cli-operate) to complete these operational steps. The Fabric SDKs are the recommended path, although the instructions assume familiarity with the operation of the SDK.

**Note**: An {{site.data.keyword.blockchainfull_notm}} Platform peer in AWS does not have access to the full functionality or support of peers that are hosted on {{site.data.keyword.blockchainfull_notm}} Platform. As a result, you cannot use the Network Monitor to operate your peer. Before you start running peers in AWS, ensure that you review the [considerations](remote_peer.html#remote-peer-limitations).

## Using Fabric SDKs to operate the peer
{: #aws-peer-operate-with-sdk}

The Hyperledger Fabric SDKs provide a powerful set of APIs that enable applications to interact with and operate blockchain networks. You can find the latest list of supported languages and the complete list of available APIs within the Hyperledger Fabric SDKs in the [Hyperledger Fabric SDK Community documentation ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/getting_started.html#hyperledger-fabric-sdks "Hyperledger Fabric SDK Community documentation"). You can use the Fabric SDKs to join your peer to a channel on the {{site.data.keyword.blockchainfull_notm}} Platform, install a chaincode on your peer, and instantiate the chaincode on a channel.

The following instructions use the [Fabric Node SDK ![External link icon](../images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/ "Hyperledger Fabric SDK for jode.js") to operate the peer and assume prior familiarity with the SDK. You can use the [developing applications tutorial](../v10_application.html) to learn how to use the Node SDK before you get started, and as a guide to developing applications with your peer once you are ready to invoke and query chaincode.

The {{site.data.keyword.blockchainfull_notm}} Platform Peer on AWS Quick Start creates two peers for high availability. Therefore, you need to follow the operations steps once for each peer. Once you are ready to query and invoke chaincode from your application, have your SDK connect to both peers to ensure your [applications are highly available](../v10_application.html#ha-app).

### Installing the Node SDK

You can use NPM to install the [Node SDK ![External link icon](../images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/ "Hyperledger Fabric SDK for jode.js"):
```
npm install fabric-client@1.2
```
{:codeblock}

It is recommended that you use version 1.2 of the Node SDK.

### Preparing the SDK to work with the peer
{: #remote-peer-node-sdk}

Before you use the SDK to operate the peer, you need to generate necessary certificates (enrollment) that will allow your application to communicate with your network on {{site.data.keyword.blockchainfull_notm}} Platform and your peer. Follow the steps to [enroll with the SDK](../v10_application.html#enroll-app-sdk) using your **admin** identity. The [Developing Applications](../v10_application.html) tutorial also enrolls as the **admin**, so you don't need to modify the sample code.

### Uploading a signcert to the IBM Blockchain Platform
{: #remote-peer-upload-SDK}

You need to upload your SDK signing cert to the network on {{site.data.keyword.blockchainfull_notm}} Platform so that other members can recognize your digital signature.

- You can find your signing cert in the directory your SDK generated your crypto material, in the file named admin. Copy the certificate inside the quotation marks after the `certificate` field, starting with  `-----BEGIN CERTIFICATE-----` and ending with `-----END CERTIFICATE-----`. You can use the CLI to convert the certificate into PEM format by issuing the command `echo -e "<CERT>" > admin.pem`. You can then paste the contents of the certificate into your blockchain network using the Network Monitor. Log in to the network on {{site.data.keyword.blockchainfull_notm}} Platform. On the "Members" screen of the Network Monitor, click **Certificates** > **Add Certificate**. Specify any name for your certificate and paste the contents in to the **Certificate** field. Click **Restart** when your are asked if you want to restart your peers. This action must be performed before your organization joins the channel.

### Uploading a signcert to the peer
{: #remote-peer-upload-signcert}

You also need to upload the SDK's signing cert into the remote peer and restart it. You need to install the same signing cert as the one that you [uploaded to the IBM Blockchain Platform](#remote-peer-upload-SDK) inside the remote peer container.

SSH into your VPC instance by selecting the instance in the AWS console (click **Services > EC2 > Instances**) and then clicking the Connect button. Follow the instructions from AWS for issuing the ssh command.

Run the following commands from inside the peer container to upload your cert.  
```
cd /etc/hyperledger/<PEER_ENROLL_ID>/msp/admincerts/
echo -e "<CERT.PEM>" > cert2.pem
```
{:codeblock}

- Replace `<PEER_ENROLL_ID>` with the enroll ID that is specified in the Quick Start template and associated with this remote peer instance.  
- Replace `<CERT.PEM>` with your signcert, which starts with `-----BEGIN CERTIFICATE-----` and ends with `-----END CERTIFICATE-----`.    

  **Note:** If the file `cert.pem` exists, do not overwrite it, but create a new file, for example, `cert2.pem`.

Because you add a new certificate, you need to restart the container for the peer to pick up the cert. Follow these [instructions](#remote-peer-aws-restart) to restart your peer.

### Passing your peer's TLS cert to the SDK
{: #aws-remote-peer-download-tlscert}

You need to copy the contents of the TLS `cacert.pem` from the peer container to your application to authenticate communication from your SDK. Run the following command in the the Peer CLI container. Replace `<PEER_ENROLL_ID>` with the stack name of the remote peer you specified in the Quick Start template. (Recall two VPC instances are created.)
```
cat /etc/hyperledger/<PEER_ENROLL_ID>/tls/ca.crt
```
{:codeblock}

Copy the entire text of the `ca.crt` file to your clipboard, which starts with the first `-----BEGIN CERTIFICATE-----` and ends with the very last `-----END CERTIFICATE-----`. Save this certificate to your local file system in PEM format. You can then import the TLS cert into your application using a simple read file command.
```
var caCert = fs.readFileSync(path.join(__dirname, './ca.pem'));
```
{:codeblock}

### Providing the peer endpoint information to the SDK
{: #remote-peer-SDK-endpoints}

To retrieve the endpoint information of your peer, locate your peer instance in the AWS console. Make note of the value of the `AWS EC2 dashboard Public DNS (IPv4)` for the EC2 instance and provide it to the SDK by declaring a new peer variable or by updating your connection profile. Concatenate the public DNS address with port `7051`. The following example defines the peer on your fabric network and passes it the TLS cert you imported.

```
var peer = fabric_client.newPeer('grpcs://<AWS_EC2_dashboard_Public_DNS>:7051', { pem:  Buffer.from(caCert).toString()});
```
{:codeblock}

**Note:** Because the peer is remote, other organizations on the {{site.data.keyword.blockchainfull_notm}} Platform network will not be able to find your peer's endpoint information in their connection profile. If other organizations need to send your peer a transaction to be endorsed, you need to provide the Public IP and TLS certs in an out of band operation.

### Using the SDK to join to a channel
{: #remote-peer-join-channel-sdk}

As a member of the blockchain network, your organization needs to be added to a channel in the network before you can join your peer to the channel.

  - You can start a new channel for the peer. As the channel initiator, you can automatically include your organization during [channel creation](create_channel.html#creating-a-channel).

  - Another member of the blockchain network can also add your organization to an existing channel using a [channel update](create_channel.html#updating-a-channel).

    After your organization is added to a channel, you need to add your peer's signing certificate to the channel so that other members can verify your digital signature during transactions. The peer uploads its signing cert during installation, so that you need to only synchronize the certificate to the channel. On the "Channels" screen of the Network Monitor, locate the channel that your organization joins and select **Sync Certificate** from the drop-down list under the **Action** header. This action synchronizes the certificates across all the peers on the channel. You might need to wait for a few minutes so that the channel sync can complete before you issue join channel commands.

After your organization is part of the channel, follow the instructions of [Joining a channel](../v10_application.html#join-channel-sdk). You need to provide the URL of the ordering service and the channel name.

### Using the SDK to install chaincode on the peer
{: #remote-peer-install-cc-sdk}

Follow the instructions to use the SDK to [install a chaincode](../v10_application.html#install-cc-sdk) on your peer.

### Using the SDK to instantiate chaincode on the channel
{: #remote-peer-instantiate-cc-sdk}

Only one member of the channel needs to instantiate or update chaincode. Therefore, any network member of the channel with peers on {{site.data.keyword.blockchainfull_notm}} Platform can use the Network Monitor to instantiate chaincode and specify endorsement policies. However, if you want to use the peer to instantiate chaincode on a channel, you can use the SDK and follow the instructions to [instantiate a chaincode](../v10_application.html#instantiate-cc-sdk).


## Using the CLI to operate the peer
{: #aws-peer-cli-operate}

You can also operate your peer from the command line using the Fabric CA client and Fabric tools container. In these instructions, we will first generate required certificates by using the Fabric CA client. We will then use the Fabric tools container to operate the peer by joining it to a channel, installing a chaincode, and then instantiating the chaincode on a channel.

### Enrollment by using Fabric CA client
{: #peer-client-enroll}

The first step is to generate the required certificates (enrollment) by using the Fabric CA client. You need to install the Fabric CA client first. Download the [fabric-ca binaries v1.2.0 for your platform](https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric-ca/hyperledger-fabric-ca/) to your local machine, extract them, and move them to a folder such as `$HOME/fabric-ca-remote/`.

1.  Prepare your environment to use the Fabric CA client. Change to the directory where you moved your client binaries so that you can reference it directly in your commands.
    ```
    cd $HOME/fabric-ca-remote/
    ```
    {:codeblock}

2.  Set the path where the client can create your keys.  If you have not run the `enroll` command before, the `msp` folder and the `.yaml` file do not exist. If you have run the `enroll` command before, it is important that you delete any configuration material from previous enroll attempts using the `rm` commands below:
    ```
    export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca-remote
    rm -rf $FABRIC_CA_CLIENT_HOME/fabric-ca-client-config.yaml
    rm -rf $FABRIC_CA_CLIENT_HOME/msp
    ```
    {:codeblock}

3.  You will need retrieve the endpoint information of your Certificate Authority on the {{site.data.keyword.blockchainfull_notm}} Platform. On the "Overview" screen of your Network Monitor, click the **Connection Profile** button and open the JSON file that contains the network credentials. Scroll down to the `certificateAuthorities` section and make a note of the information below.
    ```
    Hostname and port for CA: url (without the https prefix)
    Admin user ID: enrollId
    Admin password: enrollSecret
    CA Name: caName
    ```

4.  You will also need to retrieve a TLS certificate from the {{site.data.keyword.blockchainfull_notm}} Platform to communicate with the CA. The network credentials above include the necessary cert. In the `certificateAuthorities` section of the Connection Profile file, copy the certificate that follows `"pem:"`, which begins with `-----BEGIN CERTIFICATE-----` and ends with `-----END CERTIFICATE----- `. **Do not include the quotation marks**.

    Run the following command From a terminal window, replacing `<certificate>` with your copied certificate.
    ```
    echo -e "<certificate>" > $HOME/fabric-ca-remote/cert.pem
    ```
    {:codeblock}

 Store this cert in a place where you can reference it in future commands.  

5.  You are now ready to generate the required certificates. Running the following command:
    ```
    ./fabric-ca-client enroll -u https://<enroll_id>:<enrollSecret>@<ca_hostname_with_port> --caname <ca_name> --tls.certfiles <tls_cert_file>
    ```
    {:codeblock}

    For example:
    ```
    ./fabric-ca-client enroll -u https://admin:C25A062870@ash-zbc07c.4.secure.blockchain.ibm.com:21241 --tls.certfiles $HOME/fabric-ca-remote/cert.pem --caname PeerOrg1CA
    ```
    {:codeblock}

   When the command completes successfully, you can see a response that is similar to the following example:
    ```
    2018/06/13 09:00:45 [INFO] Created a default configuration file at
    /fabric-ca-remote/fabric-ca-client-config.yaml
    2018/06/13 09:00:45 [INFO] TLS Enabled
    2018/06/13 09:00:45 [INFO] generating key: &{A:ecdsa S:256}
    2018/06/13 09:00:45 [INFO] encoded CSR
    2018/06/13 09:00:46 [INFO] Stored client certificate at
    /fabric-ca-remote/msp/signcerts/cert.pem
    2018/06/13 09:00:46 [INFO] Stored root CA certificate
    at /fabric-ca-remote/msp/cacerts/ash-4-secure-blockchain-ibm-com-71139-PeerOrg1CA.pem
    2018/06/13 09:00:46 [INFO] Stored intermediate CA certificates at
    /fabric-ca-remote/intermediatecerts/ash-4-secure-blockchain-ibm-com-71139-PeerOrg1CA.pem
    ```

    The command will generate the required certs and store them then in a folder named `msp` in `$FABRIC_CA_CLIENT_HOME`. This folder and the certificates inside will be important in the following steps.

### Managing the certificates on your local system
{: #manage-certs}

Before we can operate the peer, we need to do some management of the certificates on our local machine, and upload some of the certs the Fabric CA client generated to the {{site.data.keyword.blockchainfull_notm}} Platform and your peer. We also need to download TLS certificates from the Platform and the peer. If you want to learn more about the certificates you will be working with and the tasks you will be performing, visit [Managing certificates on {{site.data.keyword.blockchainfull_notm}} Platform](../certificates.html).

On your local machine, open a command terminal and navigate to the directory where you moved the Fabric-CA-Client binaries and stored the MSP folder.

1. Copy the `cert.pem` file from the `signcerts` folder to a new `admincerts` folder.
   ```
   cd /$FABRIC_CA_CLIENT_HOME/msp/
   mkdir admincerts
   cd admincerts/
   cp ../signcerts/cert.pem  .
   ```
   {:codeblock}

2. Copy your orderers TLS certificate from the {{site.data.keyword.blockchainfull_notm}} Platform into the MSP folder. Your connections profile contains the necessary cert. On the "Overview" screen of your Network Monitor, click the **Connection Profile** button and open the JSON file that contains the network credentials. Navigate to the `orderers` section. Copy the certificate that follows `"pem:"`, beginning with `-----BEGIN CERTIFICATE-----` and ending with `-----END CERTIFICATE----- `. Do not include the quotation marks.

    From your terminal window, run the following commands:
    ```
    cd /$FABRIC_CA_CLIENT_HOME/msp
    echo -e "<paste in the certificate here>" > orderer_tlscacert.pem
    ```
    {:codeblock}

3. You also need to copy your peer's TLS certificate from the peer container on AWS to your local machine.

    - [Follow these instructions](remote_peer_aws.html#remote-peer-aws-test) to log in to your peer container and run the following command, replacing <PEER_ENROLL_ID> with the stack name of the peer, which you specified in the Quick Start template, followed by its number. (Recall that two VPC instances are created.)
      ```
      cat /etc/hyperledger/<PEER_ENROLL_ID>/tls/ca.crt
      ```
      {:codeblock}

      Copy the entire text of the `cacert.pem` file to your clipboard, starting with the first `-----BEGIN CERTIFICATE-----` and ending with the very last `-----END CERTIFICATE-----`.

    - Switch back to your local machine and run the following commands. Replace the text `<CACERT.PEM>` with the text from the clipboard.
      ```
      cd /$FABRIC_CA_CLIENT_HOME/msp
      mkdir tls
      cd tls/
      echo -e "<CACERT.PEM>" > cacert.pem
      ```
      {:codeblock}

      **Note:** By default, the AWS Quick Start template creates two peer instances in two different availability zones. If you have already completed these steps for one of these peers, when you run through these steps for the second instance, the cacert.pem file will already exist. Go ahead and replace it with the certificate from the second peer.

4. To give your CLI the authorization to install chaincode on the peer, upload the signCert generated by the Fabric CA client to the peer's admin folder and restart the peer. Therefore, copy the `admincert/cert.pem` certificate from your local machine into the `/etc/hyperledger/<PEER_ENROLL_ID>/msp/admincerts/` directory in the peer container as `cert2.pem`.

    - On your local machine, run the following commands:

      ```
      cd /$FABRIC_CA_CLIENT_HOME/msp/admincerts
      cat cert.pem
      ```
      {:codeblock}

    - Copy the text of the `cert.pem` file to your clipboard starting with `-----BEGIN CERTIFICATE-----` and ending with the `-----END CERTIFICATE-----`.

    - Switch back to the Peer container and run the following commands to add the cert.pem to the peer certs. Replace the text `<CERT.PEM>` with text from the clipboard.

      ```
      cd /etc/hyperledger/<PEER_ENROLL_ID>/msp/admincerts/
      echo -e "<CERT.PEM>" > cert2.pem
      ```
      {:codeblock}

      **Note:** A cert.pem file already exists in this directory. Do not overwrite it.

    - Since we added a new certificate, we need to restart the container for the peer pick up the cert. Follow these instructions to [restart the peer container](#remote-peer-aws-restart).

5. Upload the same `admincert/cert.pem` certificate from your local machine to the {{site.data.keyword.blockchainfull_notm}} Platform in order for a remote CLI or application to perform channel operations such as fetching the channel genesis block and instantiating chaincode.
    1. On your local machine, cat the file `/$FABRIC_CA_CLIENT_HOME/msp/admincerts/cert.pem` and then copy it to the clipboard.
    2. Enter the Network Monitor of your network on {{site.data.keyword.blockchainfull_notm}} Platform.
    3. Click **Members** in the left navigator and click the **Certificates** tab.
    4. Click the **Add Certificate** button.
    5. In the **Add certificate** window, give your certificate a name, such as `fabrictools.pem`, paste in the certificate you just copied to the clipboard and click **Submit**.
    6. You will be asked  if you want to restart the peers. Click **Restart**.
    7. Click **Channels** in the left navigator and locate the channel that the peer will join.
    8. Click the three dots under the **Actions** header and click **Sync Certificate**. In the **Sync certificate** window, click **Submit**.

    **Note**: It is important to sync the channel certificate before the peer joins the channel or instantiates chaincode on the channel. You might need to wait for a few minutes so that the channel sync can complete before you issue join channel or instantiate chaincode commands.   

### Setting up the Fabric tools container
{: #setup-fabric-cli}

After you move all of our certificates to the necessary location, you can install and use the Fabric tools container from Docker. Ensure that you run these commands from the directory where you stored your MSP folder.

1.  Download the Fabric tools Docker image with the following command:
    ```
    docker pull ibmblockchain/fabric-tools:1.2.0
    ```
    {:codeblock}

2.  Run the following commands to start the Fabric tools container.
    ```
    docker network create blockchain.com
    docker run -ti --network blockchain.com -v ${PWD}:/mnt ibmblockchain/fabric-tools:1.2.0
    ```
    {:codeblock}

    This command will mount the MSP directory inside your tools container.

<!--
3.  The peer address must resolve to a domain name with a suffix of `blockchain.com`. You can do this either via your DNS or modify your /etc/hosts file. For purposes of these instructions, we will modify the `/etc/hosts` file in the Fabric tools container. To retrieve the endpoint information of your peer, locate your peer instance in the AWS console. Make note of the value of the AWS `IPv4 Public IP` for the EC2 instance.

    ```
    echo -e "<AWS_IPv4_PUBLIC_IP> remotepeer.blockchain.com" >> /etc/hosts
    ```
    {:codeblock}
-->

### Using the Fabric tools CLI to join the peer to the channel
{: #fabric-cli-join-peer-to-channel}

Before you can run the CLI commands to join the peer to a channel, your organization needs to be added to a channel in the network.

  - You can start a new channel for the peer. As the channel initiator, you can automatically include your organization during [channel creation](create_channel.html#creating-a-channel).

  - Another member of the blockchain network can also add your organization to an existing channel using a [channel update](create_channel.html#updating-a-channel).

    After your organization is added to a channel, you need to add your peer's signing certificate to the channel so that other members can verify your digital signature during transactions. The peer uploads its signing cert during installation, so that you need to only synchronize the certificate to the channel. On the "Channels" screen of the Network Monitor, locate the channel that your organization joins and select **Sync Certificate** from the drop-down list under the **Action** header. This action synchronizes the certificates across all the peers on the channel.

1. Retrieve configuration information from your `Connection Profile` available in the Overview panel of the Network Monitor. Click **Connection Profile** and then **Download**.

  - Find the orderer URL by searching for **orderers**, which is located under `orderers > url`. Make a note of the URL that begins with the network name. The URL is similar to the following example:

    ```
    ash-zbc07b.4.secure.blockchain.ibm.com:21239
    ```

  - Find the name of your organization by searching for **organizations**. This should be the same organization as you use to register your peer. You can find your organization's name together with its associated `mspid`. Make a note of the value of the `mspid`.

2. Locate the endpoint information of your AWS VPC peer (the Public DNS IPv4 address) in the AWS EC2 dashboard Public DNS (IPv4). It will look something like:

  ```
  ec2-10-221-8-71.us-north-2.compute.amazonaws.com
  ```

  Use this information to build the environment variable `PEERADDR` by concatenating the address with the port '7051', so the
  value of `PEERADDR` would be similar to:

  ```
  ec2-10-221-8-71.us-north-2.compute.amazonaws.com:7051
  ```
3. Run the following commands to set the environment variables inside the Fabric tools container.

    ```
    export ORDERER_1=<ORDERER_URL>
    export CHANNEL=<CHANNEL_NAME>
    export CC_NAME=<CC_NAME>
    export ORGID=<ORGANIZATION_MSP_ID>
    export PEERADDR=<PEER_ADDR>
    ```
    {:codeblock}

    Replace the fields with your own information.
      - Replace `<ORDERER_URL>` with the hostname and port of the orderer from the `creds.json` file.
      - Replace `<CHANNEL_NAME>` with the name of the channel that the peer joins.
      - Replace `<CC_NAME>` with any name to refer to your chaincode.
      - Replace `<ORGANIZATION_MSP_ID>` with the name of the organization from the `creds.json` file.
      - Replace `<PEER_ADDR>` with the value that you constructed in the previous step.  

  For example:

    ```
    export ORDERER_1=ash-zbc07b.4.secure.blockchain.ibm.com:21239
    export CHANNEL=defaultchannel
    export CC_NAME=mycc
    export ORGID=PeerOrg1
    export PEERADDR=ec2-10-221-8-71.us-north-2.compute.amazonaws.com:7051
    ```
    {:codeblock}

3. Fetch the genesis block of the channel to build the channel ledger on your peer. First `cd` to your root directory then run the command to fetch the genesis block.

  ```
  cd /

  GOPATH=/ CORE_PEER_TLS_ROOTCERT_FILE=/mnt/msp/tls/cacert.pem CORE_PEER_TLS_ENABLED=true CORE_PEER_ADDRESS=${PEERADDR} CORE_PEER_LOCALMSPID=${ORGID} CORE_PEER_MSPCONFIGPATH=/mnt/msp/ GOPATH=/ peer channel fetch 0 -o ${ORDERER_1} -c ${CHANNEL} --cafile /mnt/msp/orderer_tlscacert.pem --tls
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

  When the genesis block has been successfully added, you should see something that is similar to the following example:
  ```
  defaultchannel_0.block
  ```
  <!-- removing the code block since this is a result and not an executable command
  {:codeblock}
  -->
4. Now join the peer to the channel, passing in the genesis block using the following command:

  ```
  GOPATH=/ CORE_PEER_TLS_ROOTCERT_FILE=/mnt/msp/tls/cacert.pem CORE_PEER_TLS_ENABLED=true CORE_PEER_ADDRESS=${PEERADDR} CORE_PEER_LOCALMSPID=${ORGID} CORE_PEER_MSPCONFIGPATH=/mnt/msp/ GOPATH=/ peer channel join -b ${CHANNEL}_0.block
  ```
  {:codeblock}

  When this works successfully you should see something similar to:

  ```
  2018-07-06 18:32:09.509 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
  2018-07-06 18:32:09.992 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
  2018-07-06 18:32:09.992 UTC [main] main -> INFO 003 Exiting.....
  ```

### Using the Fabric tools container to install chaincode on the peer
{: #aws-toolcontainer-install-cc}

We are now ready to install and instantiate chaincode on the peer. For these instructions, we'll install the `fabcar` chaincode from the `fabric-samples` repository. Download the `fabric-samples` chaincode from GitHub using the following commands:

  ```
  cd /
  mkdir /src
  cd /src
  git clone https://github.com/hyperledger/fabric-samples
  ```
  {:codeblock}

Run the following Peer CLI command to install the `fabcar` chaincode onto the peer.

  ```
  GOPATH=/ CORE_PEER_TLS_ROOTCERT_FILE=/mnt/msp/tls/cacert.pem CORE_PEER_TLS_ENABLED=true CORE_PEER_ADDRESS=${PEERADDR} CORE_PEER_LOCALMSPID=${ORGID} CORE_PEER_MSPCONFIGPATH=/mnt/msp/ GOPATH=/ peer chaincode install -n ${CC_NAME} -v v0 -p fabric-samples/chaincode/fabcar/go/
  ```
  {:codeblock}

  When this command completes successfully, you should see something similar to:

  ```
  2018-07-06 18:39:00.461 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
  2018-07-06 18:39:00.461 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
  2018-07-06 18:39:01.142 UTC [main] main -> INFO 003 Exiting.....
  ```

### Using the Fabric tools container to instantiate chaincode on a channel
{: #aws-toolcontainer-instantiate-cc}

Because only one peer has to instantiate chaincode on a channel, you do not have use your peer to instantiate chaincode. Peers hosted on {{site.data.keyword.blockchainfull_notm}} Platform can do that. However, if you would like the peer to instantiate the chaincode onto the channel, you can run the following command in the Fabric tools container:

```
CORE_PEER_TLS_ROOTCERT_FILE=/mnt/msp/tls/cacert.pem CORE_PEER_TLS_ENABLED=true CORE_PEER_ADDRESS=${PEERADDR} CORE_PEER_LOCALMSPID=${ORGID} CORE_PEER_MSPCONFIGPATH=/mnt/msp/ GOPATH=/ peer chaincode instantiate -o ${ORDERER_1} -C ${CHANNEL} -n ${CC_NAME} -v v0 -c '{"Args":[""]}' --tls --cafile /mnt/msp/orderer_tlscacert.pem -P ""
```
{:codeblock}

When this command completes successfully, you should see something similar to:

```
2018-07-06 18:43:15.066 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-07-06 18:43:15.066 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-07-06 18:43:50.227 UTC [main] main -> INFO 003 Exiting.....
```


## Restart a peer that is running in AWS
{: #remote-peer-aws-restart}

You can start, stop, or restart your peer in the environment where you deploy the peer. Whenever a certificate is added to the peer, the peer needs to be restarted in order for the new cert to be recognized. There are multiple ways to do this:

- In the AWS console, reboot the peer VPC instance.
- From inside your peer container run the following command:

 ```
 docker restart peer
 ```
 {:codeblock}  

Additionally, you can use the [HEAD request](monitor_network.html#monitor-nodes) to check the availability of your peer.

## Viewing peer logs

SSH into the AWS VPC peer instance and then run the following command to view the peer logs:

```
docker logs peer
```
{:codeblock}

## Updating chaincode

Over time it is likely that you need to modify chaincode that is running on the peer. Because chaincode is installed on the peers and instantiated on the channel, you need to update the chaincode on all of the peers on the channel.

Complete the following steps to update your chaincode:

1. To update the chaincode on each peer in AWS, simply rerun the process that you used to install the chaincode on the peers by using either a client application or a CLI command. Be sure to specify the same chaincode name as was originally used. However, this time increment the chaincode `Version` number.

2. After installing the new chaincode on all the peers in the channel, use the Network Monitor or the [peer chaincode upgrade ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/peerchaincode.html#peer-chaincode-upgrade) command to update the channel to use the new chaincode.

See step two of these [instructions](install_instantiate_chaincode.html#updating-a-chaincode) for more information on using the Install code panel of Network Monitor to update the chaincode on the channel.

## Troubleshooting

### **Problem:** Remote peer is unable to connect to blockchain network

Stack creation completes successfully, but Docker logs contain the error:

```
[main] main -> ERRO 001 Cannot run peer because error when setting up MSP of type bccsp from directory /etc/hyperledger/awspeer1/msp: Setup error: nil conf reference
```

**Solution:**  
This error can be caused by neglecting to specify a port on the CAUrl when the Quick Start template was deployed.
The CAUrl should look similar to `https://<network>-org1-ca.stage.blockchain.ibm.com:31011`.
Redeploy the Quick Start template, being careful to specify the proper value for the the CAUrl.

### **Problem:** Instantiating chaincode fails with error

Chaincode instantiation fails with:
```
Error: Error endorsing chaincode: rpc error: code = Unknown desc = chaincode error (status: 500, message: instantiation policy violation: signature set did not satisfy policy)
```

**Solution:**   
Ensure that the after the admin cert has be uploaded to the Network Monitor, that the certificates are then synchronized on the channel. Refer to step five of these [instructions](#manage-certs) for more information. Be aware that it is important to sync the channel certificate before the peer joins the channel.
