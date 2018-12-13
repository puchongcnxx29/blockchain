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

# Developing applications with the Fabric SDKs
{: #dev_app}


***[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


{{site.data.keyword.blockchainfull}} Platform provides APIs that you can use to connect applications to your blockchain network. You can use the network API endpoints in the connection profile to invoke your chaincode and update or query the channel-specific ledger on your peers. You can also use APIs in [the Swagger UI](howto/swagger_apis.html) to manage nodes, channels, and members of your network.
{:shortdesc}

You can use this tutorial to learn how to access the {{site.data.keyword.blockchainfull_notm}} Platform APIs and use them to enroll and register your application with the network. You will also learn how to interact with your network and issue transactions from your application. The tutorial is based on the [Writing Your First Application ![External link icon](images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/write_first_app.html "writing your first application"){:new_window} tutorial in the Hyperledger Fabric documentation. You will use many of the same files and commands the **Writing Your First Application** tutorial, but will use them to interact with a network on {{site.data.keyword.blockchainfull_notm}} Platform. This tutorial describes each step of application development by using the Hyperledger Fabric Node SDK. You will also learn how to enroll and register users using the Fabric CA client as an alternative to using the SDK.

In addition to this tutorial, you can use sample applications and chaincode that {{site.data.keyword.blockchainfull_notm}} Platform provides as templates when you create your own business solutions. For more information, see [Deploying sample applications](howto/prebuilt_samples.html).

## Prerequisites
You need the following prerequisites before you can use the **Writing Your First Application** tutorial on {{site.data.keyword.blockchainfull_notm}} Platform.

- If you don't have a blockchain network on {{site.data.keyword.cloud_notm}}, you need to create one with either a Starter or Enterprise Membership Plan. For more information, see [Creating a Starter Plan network](get_start_starter_plan.html#creating-a-network) or [Creating an Enterprise Plan network](get_start.html#creating-a-network).

  After you enter the Network Monitor of your network, add at least one peer for your organization on the "Overview" screen. Then, create at least one channel in your network. For more information, see [Creating a channel](howto/create_channel.html#creating-a-channel). **Note** that if you use a Starter Plan network, your network already has a channel with the name of `defaultchannel` that you can use to deploy chaincode.

- Install required tools to download Hyperledger Fabric samples and to use Node SDK.
  * [Curl ![External link icon](images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/prereqs.html#install-curl "Curl") or [Git ![External link icon](images/external_link.svg "External link icon")](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git "Git"){:new_window}
  * [Node.js ![External link icon](images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html#node-js-runtime-and-npm "Node.js"){:new_window}

- Install Hyperledger Fabric samples by downloading the `fabric-samples` directory. You can follow the [getting started guide ![External link icon](images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/install.html "getting started guide"){:new_window} in the Hyperledger Fabric documentation.

- Navigate to the `fabric-samples` directory on your local machine.
  * Use the `git checkout` command to use the branch that matches your networks Hyperledger Fabric version. You can find your Fabric version by opening the [Network Preferences window](../v10_dashboard.html#network-preferences) in your Network Monitor.
    - If your network is on Fabric version 1.2, you can use the main branch.
    - If your network is on Fabric version 1.1, run `git checkout v1.1.0`.
    - If your network is on Fabric version 1.0, run `git checkout v1.0.6`.

  * Navigate to `fabric-samples/fabcar`. You can make a copy of this directory and rename it so that you can try and test the sample application in a clean directory.

  * In the `fabcar` directory, run the `npm install` command to install the necessary packages to use the Fabric SDK, which include `fabric-client` and `fabric-ca-client`.

- Install and instantiate the fabcar chaincode on your channel by using the [Network Monitor](howto/install_instantiate_chaincode.html#installchaincode). You can find the fabcar chaincode in the `fabric-samples` folder under `fabric-samples > chaincode > fabcar > go`.

- Retrieve the Connection Profile of your network on the "Overview" screen of the Network Monitor. Save the Connection Profile into your `fabcar` directory and rename it to `creds.json`.

## Using the Fabric SDKs
{: #using-the-fabric-sdks}

The Hyperledger Fabric SDKs provide a powerful set of APIs that enable applications to interact with blockchain networks. You can find the latest list of supported languages in the [Hyperledger Fabric SDK Community documentation ![External link icon](images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/getting_started.html#hyperledger-fabric-sdks "Hyperledger Fabric SDK Community documentation"){:new_window}. It is recommended to use the Node SDK or the Java SDK with {{site.data.keyword.blockchainfull_notm}} Platform. You can learn more about the APIs that the SDKs provide in the SDKs' individual repositories.

This tutorial uses the [Node SDK ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/ "Node SDK"){:new_window} to register and enroll your application, and then use the application to issue transactions by invoking and querying chaincode. This tutorial describes the information that you need to provide to the SDK so that your application can connect to your blockchain network. It also introduces some of the APIs that you can use, and how the SDK interacts with and submits transactions to your the blockchain network.

## Adding network API endpoints to your application
{: #api-endpoints}

You need to provide your application with the API endpoints of specific network resources, including orderer, CA, and peer nodes, in the blockchain network on {{site.data.keyword.cloud_notm}}. Your application can interact with the network through these API endpoints. You can find the API endpoints in the Connection Profile of your network. The Connection Profile is in JSON format and contains the API endpoint information and enrollIDs/secrets for your network resources.

1. Retrieve the API endpoint information of your network resources from your Network Monitor with one of the following methods:
  * On the "Overview" screen, click **Connection Profile**. The Connection Profile contains a complete set of API endpoint information of all your network resources.
    ![Connection Profile in Network Monitor](images/service_credentials.png "Connection Profile in Network Monitor")

  * If you have running chaincode in the network, you can get the API endpoint information that is specific to the chaincode. On the "Channels" screen, click the the channel row on which the chaincode is running to open the specific channel screen. Then, locate the chaincode and click the **JSON** button.
    ![API endpoints per chaincode](images/channel_chaincode.png "API endpoints per chaincode")

2. Locate the API endpoint information of your network resources, which is similar to the URL of `peer1-org1` row in the following example:
  ```
    "peers": {
        "org1-peer1": {
            "url": "grpcs://n7413e3b503174a58b112d30f3af55016-org1-peer1.us3.blockchain.ibm.com:31002",
            "eventUrl": "grpcs://n7413e3b503174a58b112d30f3af55016-org1-peer1.us3.blockchain.ibm.com:31003",
                  ...
  ```

  **Note**: You might want to target network resources outside of your organization with your application. For example, if a chaincode [endorsement policy](howto/install_instantiate_chaincode.html#endorsement-policy) requires endorsements from other organizations on the channel, then you need to obtain the endpoint information of their peers and the accompanying TLS certificates. You can find this information in the peers section of Connection Profile. However, you need to contact the admin of the other orgs about which peers they have joined to particular channels.

3. Plug the API endpoint information into a configuration file of your application as shown in the following example:
  ```
  grpcs://n7413e3b503174a58b112d30f3af55016-orderer.us3.blockchain.ibm.com:31001
  ```

  You can also send [HEAD requests](howto/monitor_network.html#monitor-nodes) to these endpoints to check the availability of your network resources.

  If you are using the Fabric SDKs, you can also connect to your network using your Connection Profile. This tutorial provides the endpoint information of your network to the SDK manually. However you can find a tutorial and guidance on [using your connection profile with the SDK](#using-your-connection-profile-with-the-sdk) in a later section.

## Enrolling your application
{: #enroll-app}

Before you connect an application to your network on {{site.data.keyword.blockchainfull_notm}} Platform, you need to prove the authenticity of your application to the network. We won't delve into the details of x509 certificates and public key infrastructure, though you can learn more by visiting the [Managing certificates on {{site.data.keyword.blockchainfull_notm}} Platform](certificates.html) tutorial. Simply speaking, the communication flows in Fabric use sign/verify operations at every touchpoint. Therefore, any application that sends calls, such as ledger queries or updates, to the network needs to sign payloads with their private key and attach a properly signed x509 certificate for verification purposes. **Enrollment** is the process of generating the necessary keys and certs from the appropriate Certificate Authority. After the enrollment, your application is ready to communicate with the network.

This section explains how to retrieve the keys and certs with the Fabric Node SDK by using the sample code that is part of the **Writing Your First Application** tutorial. You can only generate certificates using an identity that has been registered with your Certificate Authority. The tutorial below first enrolls using an admin identity that has already been registered with your CA. It then uses those certificates to register a new client identity. The tutorial enrolls again using the new identity, and uses those certificates to submit transactions to the network. <!---You can find an illustration of how the developing applications tutorial interacts with your organization CA in the diagram below.--->

You can also use the "Certificate Authority" screen of the Network Monitor to generate certificates, and use those certificates to interact with the network. To learn how, visit [Generating certificates using the Network Monitor](#enroll-panel). You can also learn how to use the [Fabric CA client](certificates.html#enroll-register-caclient) from the command line to generate certificates and register users in the [Managing certificates](certificates.html) tutorial.

### Enrollment using the Fabric SDK
{: #enroll-app-sdk}

From the `fabcar` directory in your `fabric-samples` folder, open the `enrollAdmin.js` file in a text editor.

1. The file firstly creates an instance of the Fabric client.
  ```
  var fabric_client = new Fabric_Client();
  ```
  {:codeblock}

2. The file then creates a key-value store (KVS) to manage your certificates. The SDK uses the [KeyValueStore ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/module-api.KeyValueStore.html "KeyValueStore"){:new_window}  class to create the key-value store, and the [CryptoSuite ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/module-api.CryptoSuite.html "CryptoSuite"){:new_window} class to perform the crypto computations. You can see the relevant code block below.
  ```
  # create the key value store as defined in the fabric-client/config/default.json 'key-value-store' setting
  Fabric_Client.newDefaultKeyValueStore({ path: store_path
    }).then((state_store) => {
   // assign the store to the fabric client
   fabric_client.setStateStore(state_store);
   var crypto_suite = Fabric_Client.newCryptoSuite();
   // use the same location for the state store (where the users' certificate are kept)
   // and the crypto store (where the users' keys are kept)
   var crypto_store = Fabric_Client.newCryptoKeyStore({path: store_path});
   crypto_suite.setCryptoKeyStore(crypto_store);
   fabric_client.setCryptoSuite(crypto_suite);
   var	tlsOptions = {
     trustedRoots: [],
     verify: false
   };
  ```
  {:codeblock}

3. After the definition of the KVS, you can use a few methods from the [Fabric Client ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/Client.html "Fabric Client"){:new_window} class and Fabric-CA-Client API <!---[FabricCAServices ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/FabricCAServices.html "FabricCAServices")---> to communicate with the CA Server. You need to provide the SDK the name and URL of your Certificate Authority. Open the **Connection Profile** JSON file from your **Overview** screen in the Network Monitor, and find the following variables under the `certificateAuthorites` section:
  * URL for CA: ``url`` under `certificateAuthorities`
  * Admin user ID: ``enrollId``
  * Admin password: ``enrollSecret``
  * CA name: `caName`

  **Edit** the relevant lines in the `enrollAdmin.js` file with this information in the following way:
  ```
  fabric_ca_client = new Fabric_CA_Client('https://<enrollID>:<enrollSecret>@<ca_url_with_port>', null ,<caName>, crypto_suite);
  ```
  {:codeblock}

  For example:
  ```
  fabric_ca_client = new Fabric_CA_Client('https://admin:dda0c53f7b@n7413e3b503174a58b112d30f3af55016-org1-ca.us3.blockchain.ibm.com:31011', null ,'org1CA', crypto_suite);
  ```

  The Fabric client then checks whether your application is already enrolled. **Edit** the line below with the `enrollID` from your connections profile:
  ```
  return fabric_client.getUserContext('<enrollID>', true);
  ```
  {:codeblock}

4. You need to send the "enroll" call to the CA Server. Your `admin` is already registered with your network. The enroll call retrieves a private key and a public key wrapped in an x509 cert and signed by the targeted CA. This wrapped and signed cert is called the signCert. The signCert allows network members to verify calls that originate from the client. You need to provide your org name, password, and MSP ID from your credentials file. In the `certificateAuthorities` section of your network credentials, find your `enrollID`, `enrollSecret`, and `x-mspid`. **Edit** the code block below with those values and replace the relevant section of your file.
  ```
  return fabric_ca_client.enroll({
    enrollmentID: '<enrollID>',
    enrollmentSecret: '<enrollSecret>'
      }).then((enrollment) => {
    console.log('Successfully enrolled admin user');
    return fabric_client.createUser(
         username: 'admin',
            mspid: '<x-mspid>',
            cryptoContent: { privateKeyPEM: enrollment.key.toBytes(), signedCertPEM: enrollment.certificate }
        });
  ```
  {:codeblock}

5. Save the `enrollAdmin.js` file.

In the `fabcar` directory, enroll the admin by issuing the following command:
```
node enrollAdmin.js
```
{:codeblock}

The enrollment command generates the signCert and exports it into a folder named `hfc-key-store`. The future files in this tutorial will look for your certificates in this folder. If you can find the admin certs in the `hfc-key-store` folder, the enroll command works.

If you want to [operate your network using the SDK](#operate-sdk), you need to upload your admin signCert to the {{site.data.keyword.blockchainfull_notm}} Platform. You can find your admin signCert in the `hfc-key-store` folder. Open the `admin` file, and copy the certificate inside the quotation marks after the `certificate` field. Use to a tool or text editor to convert the certificate into PEM format. You can then upload the admin certificate to your blockchain network from the Network Monitor. For more information about adding certificates, see [the "Certificates" tab of "Member" screen](v10_dashboard.html#members) in the Network Monitor. This is not necessary if you are only using the SDK to invoke or query chaincode.

## Registering your application
{: #register-app}

After generating the client side certificates, you need to register your application with the network Certificate Authority. Registration adds your application to the list of components that the network can recognize. A best practice is to register your application as a separate identity rather than use the `admin` to sign requests.

### Registering using the SDK
{: #register-app-sdk}

You can use the `registerUser.js` file to register and enroll the application as `user1` by using your `admin` signCert. Open `registerUser.js` in a text editor.

1. Provide the CA URL and name to a new instance of the Fabric CA client.

2. Provide the enrollID to the `getUserContext` method from the [Fabric Client class ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/Client.html "Fabric Client class"){:new_window} to check whether your `admin` is enrolled and allowed to issue this request. **Edit** relevant code block in your file based on the sample below:
  ```
  // be sure to change the http to https when the CA is running TLS enabled
  fabric_ca_client = new Fabric_CA_Client('https://admin:dda0c53f7b@n7413e3b503174a58b112d30f3af55016-org1-ca.us3.blockchain.ibm.com:31011', null , '<caName>', crypto_suite);

  // first check to see if the admin is already enrolled
  return fabric_client.getUserContext('admin', true);
  }).then((user_from_store) => {
  if (user_from_store && user_from_store.isEnrolled()) {
      console.log('Successfully loaded admin from persistence');
      admin_user = user_from_store;
  } else {
      throw new Error('Failed to get admin.... run enrollAdmin.js');
  }
  ```
  {:codeblock}

3. Use the **Fabric CA client** to register and enroll the user with the CA, and then use the **Fabric client** to create the new signCert. **Edit** the following block with your MSP ID and your organization affiliation. You can find your `x-affiliations` in the certificate authorities section of your network credentials and use any affiliation listed. Add the name of the user that you want to create. The fabcar sample uses `user1` by default.
  ```
  return fabric_ca_client.register({enrollmentID: 'user1', affiliation: '<x-affiliations>',role: 'client'}, admin_user)
    ...
  return fabric_ca_client.enroll({enrollmentID: 'user1', enrollmentSecret: secret});
  }).then((enrollment) => {
  console.log('Successfully enrolled member user "user1" ');
  return fabric_client.createUser(
   {username: 'user1',
   mspid: '<x-mspid>',
   cryptoContent: { privateKeyPEM: enrollment.key.toBytes(), signedCertPEM: enrollment.certificate }
   });
  ```
  {:codeblock}

4. Save the `registerUser.js` file.

Run the `node registerUser.js` command to register and enroll `user1`. If you can find the `user1` certificates in the `hfc-key-store` folder, the command works. You can only register an identity once. If you experience a problem, try running `registerUser.js` with a new user name.

### Registering using the Network Monitor

Alternatively, you can register and enroll your client application using the Network Monitor **Certificate Authority** tab. Refer to this [information](v10_dashboard.html#ca) for more instructions.

## Issuing transactions by invoking and querying chaincode
{: #invoke-query}

Your application needs to interact with the full blockchain network to submit a transaction.

1. The application sends a transaction proposal to be endorsed by peers on the channel.
2. Endorsing peers return endorsed transaction to the application.
3. The application sends the endorsed transaction to the ordering service to add the transaction to the ledger.

For more information about the complete transaction flow, see [Transaction Flow ![External link icon](images/external_link.svg "External link icon")]( https://hyperledger-fabric.readthedocs.io/en/release-1.2/txflow.html "Transaction Flow"){:new_window} in the Hyperledger Fabric documentation. After you getting started with this tutorial, visit the [application connectivity and availability](#app-connectivity-availability) section for tips on managing how your SDK interacts with the network.

The following samples demonstrates how the Node SDK sets up the network topology, defines the transaction proposal, and then submits the transaction to the network. You can use the `invoke.js` file to invoke functions within the `fabcar` chaincode. These functions allow you to create and transfer assets on the blockchain ledger. This tutorial uses the `initLedger` function to add new data to your channel, and then uses the `query.js` file to query the data.

### Invoking chaincode
{: #invoke}

Open the `invoke.js` file in a text editor.

1. Add `var creds = require('./creds.json')` to the top of the file. This code line allows the `invoke.js` file to read information from the `creds.json` credentials file.

2. Use the [Fabric Client ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/Client.html "Fabric Client"){:new_window} class to set up the fabric network by using the API endpoints of your network resources. This step defines the channel and peers that your client will submit the proposal to, and the ordering service that the SDK will then send the endorsed transaction to. **Edit** the relevant code block below. The line `creds.peers["org1-peer1"].url` imports the peer url from your credentials file.
  ```
  # setup the fabric network
  var channel = fabric_client.newChannel('defaultchannel');
  var peer = fabric_client.newPeer(creds.peers["org1-peer1"].url, { pem: creds.peers["org1-peer1"].tlsCACerts.pem , 'ssl-target-name-override': null});
  channel.addPeer(peer);
  var order = fabric_client.newOrderer(creds.orderers.orderer.url, { pem: creds.orderers.orderer.tlsCACerts.pem , 'ssl-target-name-override': null})
  channel.addOrderer(order);
  ```
  {:codeblock}

  The new peer and orderer variables open GRPC connections to your blockchain network. For more information about managing these connections, see [Opening and closing network connections](#connections).

  When you add the peer URL to the `fabric_client.newPeer` method, you also import the relevant TLS certificates from your connection profile by using the code snippet below. You did the same when you added the ordering service URL. You need to use these TLS certs to authenticate communication with your network.
  ```
  { pem: creds.peers["org1-peer1"].tlsCACerts.pem , 'ssl-target-name-override': null}
  ```
  {:codeblock}

  If the endorsement policy requires that transactions be endorsed by other organizations in the channel, you need to add those organizations peers using the `newPeer()` and `channel.addPeer()` methods when you set up the network. The organizations send need to send you the list of the peers they have joined to a particular channel. The endpoint information and TLS certs will be available in the connection profile. The SDK will send the transaction to all the peers added to the channel.

  You can also add additional peers belonging to your organization that are joined to the channel as a step to [making your application highly available](#ha-app). This will provide the SDK with a failover in the event one of your peers is disrupted.

3. After setting up the fabric network, and importing the application identity and signCert from the registration step, the `invoke.js` file defines the proposal that you will submit to the network. You can use the `initLedger` function in the `fabcar` chaincode to add some initial data to your ledger. You can also edit the code block to invoke other functions that you can find in the `fabcar` chaincode.
  ```
  var request = {
    //targets: let default to the peer assigned to the client
    chaincodeId: 'fabcar',
    fcn: 'initLedger',
    args: [''],
    chainId: 'mychannel',
    txId: tx_id
  };
  ```
  {:codeblock}

  After defining the request, you can send a [transaction proposal ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/Channel.html#sendTransactionProposal "sendTransactionProposal") to the peers on the channel. After the proposal is returned from the peers, you can [send the transaction ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/Channel.html#sendTransaction "sendTransaction") to the ordering service.

4. You can add an event service to make the transaction flow more efficient. **Edit** the following section:
  ```
  let event_hub = fabric_client.newEventHub();
  event_hub.setPeerAddr(creds.peers["org1-peer1"].eventUrl, { pem: creds.peers["org1-peer1"].tlsCACerts.pem , 'ssl-target-name-override': null});
  ```
  {:codeblock}

  Although the sample uses a peer based event service, you should use a channel based listener. You can learn more in the [managing transactions](#managing-transactions) section and the [Node SDK documentation ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/tutorial-channel-events.html "channel based event service"){:new_window}.

5. By default, `invoke.js` submits the transaction as `user1`. You can edit the `invoke.js` file if you registered a different name.

After you finish editing the file, issue the `node invoke.js` command to submit the transaction to the network. You should see the following as a confirmation that the invocation was successful.
```
Successfully sent Proposal and received ProposalResponse: Status - 200, message - "OK"
```

This indicates that your application successfully invoked your chaincode and added data to the ledger.

### Querying chaincode
{: #query}

Now you can use `query.js` to read the ledger. Open the `query.js` file in a text editor.

1. Add `var creds = require('./creds.json')` to the top of the file.
2. Update the file with the channel name and the endpoint information of the peer. Because this operation reads only data that is stored on the peer, you do not need to add the endpoint information of the ordering service. `query.js` also assumes that you send the proposal as `user1`.
  ```
  var channel = fabric_client.newChannel('mychannel');
  var peer = fabric_client.newPeer(creds.peers["org1-peer1"].url, { pem: creds.peers["org1-peer1"].tlsCACerts.pem , 'ssl-target-name-override': null});
  channel.addPeer(peer);
  ```
  {:codeblock}

After you finish editing the file, issue the `node query.js` command and you should see the list of cars on your ledger, which is similar to the following example.
```
Query has completed, checking results
Response is  
  [{"Key":"CAR0", "Record":{"colour":"blue","make":"Toyota","model":"Prius","owner":"Tomoko"}},
  {"Key":"CAR1", "Record":{"colour":"red","make":"Ford","model":"Mustang","owner":"Brad"}},
  ...
```
{:codeblock}

For more information about the fabcar application and the functions it uses, you can visit the full tutorial of [Writing Your First Application ![External link icon](images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/write_first_app.html "writing your first application"){:new_window} in the Hyperledger Fabric documentation.

## Using your Connection Profile with the SDK
{: #using-your-connection-profile-with-the-sdk}

Instead of importing the endpoint information of your network manually, you can have the SDK connect to your network by using the **Connection Profile** from the **Overview** screen of your Network Monitor. This streamlines the process of connecting to your certificate Authority for enrollment and registration. It also eliminates the need to define your fabric network before you submit a transaction. The SDK will find the peers and orderers on the relevant channel directly from the Connection Profile. You can find more information on how to use a Connection Profile in the [Node SDK documentation ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/tutorial-network-config.html "connection profile tutorial"){:new_window}.

We can use the `invoke.js` file as an example to see the efficiency of using a Connection Profile rather than manual endpoints. You can establish a new instance of the the fabric client using the `loadFromConfig` class. Replace `var fabric_client = new Fabric_Client();` with the following code.
```
var fabric_client = Fabric_Client.loadFromConfig(path.join(__dirname, './connection-profile.json'));
```
{:codeblock}

Instead of setting up the fabric network by creating peers and orderers, and then adding them to a channel, you can use the following line to create a new channel.

```
var channel = fabric_client.newChannel('defaultchannel');
```
{:codeblock}

The SDK then adds the peers and ordering service that are defined on the channel using the Connection Profile. This makes writing applications more efficient, and makes it easier to update your applications as network members join, leave, and start new channels. Please refer to the [connection profile tutorial ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/tutorial-network-config.html "connection profile tutorial"){:new_window} in the Node SDK Documentation to learn about the additional steps involved. You can use of this [version of the fabcar tutorial ![External link icon](images/external_link.svg "External link icon")](https://www.ibm.com/developerworks/cloud/library/cl-deploy-fabcar-sample-application-ibm-blockchain-starter-plan/index.html){:new_window} that uses a connection profile rather than manual endpoint connections.

You can send transactions to peers that are outside of your organization for endorsement by editing your Connection Profile. The Connection Profile already contains the endpoint information and TLS certificates of peers from other organizations on your {{site.data.keyword.blockchainfull_notm}} Platform network. Add the name of the peer to the relevant channel in the `channels` section of the profile to add the peer to the channel.

## Generating certificates by using the Network Monitor
{: #enroll-panel}

You can use the Network Monitor to generate certificates by using the admin identity, and then pass those certificates directly to the SDK. This means that you can get started interacting with the network quickly, without having to generate certificates using the SDK.

Navigate to the "Certificate Authority" panel in your Network Monitor. Click the **Generate Certificate** button next to your admin identity to get a new signCert and private key from your CA. The **Certificate** field contains the signCert, just above the **Private Key**. You can click the copy icon at the end of each field to copy the value. Save these certificates place where you can provide them to your application. **Note** that {{site.data.keyword.blockchainfull_notm}} Platform doesn't store these certificates. You need to safely save and store them.

The signCert and private key are sufficient to form a user context that can sign requests within the Node SDK. Use the [createUser ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/Client.html#createUser__anchor "create user"){:new_window} method of the Client class to create the user context object. Within the `creatUser` method, pass the identity name and mspid to the [user ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/global.html#UserOpts "user"){:new_window} object, along with the paths to the private key and signCert to the [cryptoContent ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/global.html#CryptoContent "crypto content"){:new_window} object.

We can use the "Certificate Authority" panel and the `createUser` class as part of the developing applications tutorial as an example. If you have already gone through the tutorial, then you have installed and instantiated the `fabcar` chaincode, and added some data to your ledger. We can use the certificates to query the ledger as the `admin` user. Follow the instructions to generate certificates using the Network Monitor above if you have not already done so.

Save your private key as privateKey.pem and your signCert as certificate.pem in the same directory as `query.js`. Open `query.js` in a text editor. Add the following line to the top of the file:
```
var fs = require('fs');
```
Replace the following line importing the user context from persistence,
```
return fabric_client.getUserContext('user1', true);
```
with the following code creating a new user context using our signCert and private key.
```
return fabric_client.createUser({
		username: 'admin',
		mspid:  'org1',
		cryptoContent: {
			privateKeyPEM: fs.readFileSync(path.join(__dirname,'./privateKey.pem')),
			signedCertPEM: fs.readFileSync(path.join(__dirname,'./certificate.pem'))
		}});
```
The snippet above reads your certificates directly to the `cryptoContent` class as PEM files. The username will be `admin`, since the certificates were generated using the `admin` identity. You can find your mspid in the `certificateAuthorites` section of your connection profile. Save the file and issue the command `node query.js`. If successful, the query will return the same results as before.

## Best practices for application connectivity and availability
{: #app-connectivity-availability}

The Hyperledger Fabric [Transaction Flow ![External link icon](images/external_link.svg "External link icon")]( https://hyperledger-fabric.readthedocs.io/en/release-1.2/txflow.html "Transaction Flow"){:new_window} spans multiple components, with the client applications playing a unique role. The SDK submits transaction proposals to the peers for endorsement. It then collects the endorsed proposals to be sent to the ordering service, which then sends blocks of transactions to the peers to be added to channel ledgers. Developers of production applications should be prepared to manage their interactions between the SDK and their networks for efficiency and availability.

### Managing transactions
{: #managing-transactions}

Application clients should ensure that their transaction proposals are validated and that the proposals complete successfully. A proposal can be delayed or lost for multiple reasons, such as a network outage or a component failure. You should prepare your application for [high availability](#ha-app) to handle component failure. You can also [increase the timeout values](#set-timeout-in-sdk) in your application to prevent proposals from timing out before the network can respond.

If a chaincode is not running, the first transaction proposal that is sent to this chaincode will start the chaincode. While the chaincode is starting, all other proposals are rejected with an error that indicates that the chaincode is currently starting. This is different from transaction invalidation. If any proposal is rejected while the chaincode is starting, application clients need to send the rejected proposals again after the chaincode starts. Application clients can use a message queue to avoid losing transaction proposals.

You can use a channel-based event service to monitor transactions and build message queues. The [channelEventHub ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/ChannelEventHub.html "channelEventHub"){:new_window} class can register listeners based on transaction, block, and chaincode events. Channel-based listeners from the channel eventhub can scale to multiple channels and distinguish between traffic on different channels.

It is recommended that you use channelEventHub rather than the old eventHub class. EventHub is single threaded and contains events from all channels that could slow down or even hang listeners across channels. The eventHub class also provides no guarantee that an event will be delivered, and provides no way of retrieving events from a certain point, such as a block number, to track events that were missed.

**Note** that the peer eventhub will be deprecated in a future release of the Fabric SDK. If you have existing applications that use the peer eventhub, update your applications to use the channel eventhub. For more information, see [How to use the channel-based event service ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/tutorial-channel-events.html "How to use the channel-based event service"){:new_window} in the Node SDK Documentation.

### Opening and closing network connections
{: #connections}

When you create peer and orderer objects with the SDK before submitting transaction proposals, you are opening a gRPC connection between your application and the network component. For example, the following command opens a connection to `org1-peer1`. This connection continues to be active while your application is running.

```
var peer = fabric_client.newPeer(creds.peers["org1-peer1"].url, { pem: creds.peers["org1-peer1"].tlsCACerts.pem , 'ssl-target-name-override': null});
```
{:codeblock}

When you manage the connections between your application and your network, you might consider the following recommendations.

- Reuse peer and orderer objects when you interact with your network, instead of opening new connections to submit transactions. Reusing peer and orderer objects can save resources and lead to better performance.  
- To maintain a persistent connection to your network components, use [gRPC keepalives ![External link icon](images/external_link.svg "External link icon")](https://github.com/grpc/grpc/blob/master/doc/keepalive.md "gRPC Keepalives"). Keepalives keep the gRPC connection active and prevent an "unused" connection from being closed. The following example of peer connection adds gRPC options to the [Connection Options ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/global.html#ConnectionOpts "Connection") object. The gRPC options are set to values that {{site.data.keyword.blockchainfull_notm}} Platform recommends.  
  ```
  var peer = fabric_client.newPeer(creds.peers["org1-peer1"].url, { pem: creds.peers["org1-peer1"].tlsCACerts.pem , 'ssl-target-name-override': null},
  "grpcOptions": {
    "grpc.keepalive_time_ms": 120000,
    "grpc.http2.min_time_between_pings_ms": 120000,
    "grpc.keepalive_timeout_ms": 20000,
    "grpc.http2.max_pings_without_data": 0,
    "grpc.keepalive_permit_without_calls": 1
    }
  );
  ```
  {:codeblock}

  You can also find these variables with the recommended settings in the `"peers"` section of your network connection profile. The recommended options will be imported into your application automatically if you use the [connection profile with the SDK](#using-your-connection-profile-with-the-sdk) to connect to your network endpoints.

- If a connection is no longer needed, use the `peer.close()` and `orderer.close()` commands to free up resources and prevent performance degradation. For more information, see the [peer close ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/Peer.html#close__anchor "peer close") and [orderer close![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/Orderer.html#close__anchor "orderer close") classes in the Node SDK documentation. If you used a connection profile to add peers and orderers to a channel object, you can close all connections that are assigned to that channel by using a `channel.close()` command.

### Highly available applications
{: #ha-app}

As a high availability best practice, it is strongly recommended that you deploy a minimum of two peers per organization for failover. You need to adapt your applications for high availability as well. Install chaincode on both peers and add them to your channels. Then be prepared to [submit transaction proposals](#invoke) to both peer endpoints when setting up your network and building your peer target list. Enterprise Plan networks have multiple orderers for failover, which allows your client application to send endorsed transactions to a different orderer if one orderer is not available. If you use your [connection profile](#using-your-connection-profile-with-the-sdk) instead to add network endpoints manually, ensure that your profile is up to date and that the additional peers and orderers have been added to the relevant channel in the `channels` section of the profile. The SDK can then add the components that are joined on the channel by using the connection profile.

## Enabling mutual TLS
{: #mutual-tls}

If you are running Enterprise Plan networks that is at Fabric V1.1 level, you have the option of [enabling mutual TLS](v10_dashboard.html#network-preferences) for your applications. If you enable mutual TLS, you need to update your applications to support this function. Otherwise, your applications cannot communicate with your network.

In the Connection Profile, locate the `certificateAuthorities` section where you can find the following attributes that are necessary to enroll and get the certificates to communicate with your network by using mutual TLS.

- `url`: URL for connecting to the CA that can give out mutual TLS certificates
- `enrollId`: Enroll ID to use for getting a certificate
- `enrollSecret`: Enroll secret to use for getting a certificate
- `x-tlsCAName`: CA name to use for getting certificate that allows the application to communicate with Mutual TLS.

For more information about updating your applications to support mutual TLS, see [How to configure mutual TLS ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/tutorial-mutual-tls.html "mutual tls"){:new_window}.

## (Optional) Operating your network using the SDK
{: #operate-sdk}

You can also use the SDK to operate your blockchain network. This tutorial explains how you can use the SDK to join your peers to channels, install chaincode on your peers, and instantiate chaincode on channels. These steps are optional because you can also perform these operations using the Network Monitor or the APIs in the [Swagger UI](howto/swagger_apis.html) if all of your peers are being run on the {{site.data.keyword.blockchainfull_notm}} Platform.

You need to upload your admin signCert to the {{site.data.keyword.blockchainfull_notm}} Platform in order to complete these steps. You can find instructions on how to upload your signCert at the end of the [enrollment section](#enroll-app-sdk)

### Joining a channel
{: #join-channel-sdk}

After your organization creates or joins a channel by using the Network Monitor or the APIs, you can use the SDK to join your peer to the channel.

1. [Fetch the genesis block ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/Channel.html#getGenesisBlock "fetch genesis block"){:new_window} of the channel from the ordering service.
2. Pass the genesis block to the [join channel ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/Channel.html#joinChannel "joinChannel"){:new_window}  method to join your peer to the channel.

To use the `fabcar` sample to join a channel, use the `invoke.js` file as your starting point. You need to send this request as the admin rather than the application, so replace `user1` with `admin` in the `getUserContext` method. Starting where you define your chaincode invoke request at `var request = {`, replace the transaction flow with a channel join request based on the code snippet below from the [node SDK documentation ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/tutorial-channel-create.html "mutual tls"){:new_window}.
  ```
  let g_request = {
    txId :     tx_id
  };

  // get the genesis block from the orderer
  channel.getGenesisBlock(g_request).then((block) =>{
    genesis_block = block;
    tx_id = client.newTransactionID();
    let j_request = {
      targets : targets,
      block : genesis_block,
      txId : tx_id
    };
    // send genesis block to the peer
    return channel.joinChannel(j_request);
  }).then((results) =>{
    if(results && results[0].response && results[0].response.status == 200) {
    // good  
    } else {
      // not good
    }
  });
  ```

Your signCert need to be added to the channel before you can fetch the genesis block. If you generated certificates after your organization joined the channel, you need to upload your signCert to the platform and then click the **Sync Certificates** button in the "Channels" screen. You might need to wait a few minutes for channel sync to complete before issuing the join channel command. For more information, see [Uploading signing certificates to {{site.data.keyword.blockchainfull_notm}} Platform](certificates.html#upload-certs) in the [Managing certificates](certificates.html) tutorial.

### Installing a chaincode
{: #install-cc-sdk}

You can use the [install chaincode ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/Client.html#installChaincode "installChaincode"){:new_window} method from the [Fabric Client ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/Client.html "Fabric Client"){:new_window} class to install chaincode on your peer.

To use the `fabcar` sample to install the `fabcar` chaincode on your peer, use the `query.js` file as a baseline and edit it. You need to send this request as the admin rather than the application, so replace `user1` with `admin` in the `getUserContext` method. Replace the transaction proposal object with an install chaincode request using the example below:
```
var request = {
    targets: peer,
    chaincodePath: chaincode_path,
    chaincodeId: 'fabcar',
    chaincodeType: 'golang',
    chaincodeVersion: 'v1',
    channelNames: 'mychannel'
	 };
```
{:codeblock}

Send this object to `return fabric_client.installChaincode(request);`, instead of the `return channel.queryByChaincode(request);` line currently in the file.

### Instantiating a chaincode
{: #instantiate-cc-sdk}

To instantiate the chaincode, you need to send an [instantiate proposal ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/Channel.html#sendInstantiateProposal "sendInstantiateProposal"){:new_window} to the peer, and then send a [transaction request ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/Channel.html#sendTransaction "sendTransaction){:new_window} to the ordering service.

To use the `fabcar` sample to instantiate your chaincode, use the `invoke.js` file as your starting point. You need to send this request as the admin rather than the application, so replace `user1` with `admin` in the `getUserContext` method. Replace the transaction proposal object with an install chaincode request using the example below:
```
var request = {
    targets: peer,
    chaincodePath: chaincode_path,
    chaincodeId: 'fabcar',
    chaincodeType: 'golang',
    chaincodeVersion: 'v1',
    channelNames: 'mychannel',
    txId : tx_id
};
```
{:codeblock}

Send this request to `return channel.sendInstantiateProposal(request);` instead of the `return channel.sendTransactionProposal(request);` currently in the file. After sending the instantiate request to the channel, you then need to send the endorsed proposal as a transaction to the ordering service. This uses the same methods as sending a transaction, so you can keep the rest of the file unchanged. You may want [increase the timeout value](#set-timeout-in-sdk) in the instantiate proposal. Otherwise the request may timeout before the platform can start the chaincode container.

Your signCert needs to be added to the channel before you can instantiate chaincode. If you generated certificates after you joined the channel, you need to upload your signCert to the platform and then click the **Sync Certificates** button in the "Channels" screen. You may need to wait a few minutes for channel sync to complete before issuing the instantiate chaincode command. To learn more, see [Uploading signing certificates to {{site.data.keyword.blockchainfull_notm}} Platform](certificates.html#upload-certs) in the [Managing certificates](certificates.html) tutorial.

## (Optional) Setting timeout values in Fabric SDKs
{: #set-timeout-in-sdk}

Fabric SDKs set default timeout values in client applications for events in the blockchain network. See the following example about default timeout settings in Fabric Java SDK. The file path is `src\main\java\org\hyperledger\fabric\sdk\helper\Config.java`.

```
    /**
     * Timeout settings
     **/
    public static final String PROPOSAL_WAIT_TIME = "org.hyperledger.fabric.sdk.proposal.wait.time";
    public static final String CHANNEL_CONFIG_WAIT_TIME = "org.hyperledger.fabric.sdk.channelconfig.wait_time";
    public static final String TRANSACTION_CLEANUP_UP_TIMEOUT_WAIT_TIME = "org.hyperledger.fabric.sdk.client.transaction_cleanup_up_timeout_wait_time";
    public static final String ORDERER_RETRY_WAIT_TIME = "org.hyperledger.fabric.sdk.orderer_retry.wait_time";
    public static final String ORDERER_WAIT_TIME = "org.hyperledger.fabric.sdk.orderer.ordererWaitTimeMilliSecs";
    public static final String PEER_EVENT_REGISTRATION_WAIT_TIME = "org.hyperledger.fabric.sdk.peer.eventRegistration.wait_time";
    public static final String PEER_EVENT_RETRY_WAIT_TIME = "org.hyperledger.fabric.sdk.peer.retry_wait_time";
    public static final String EVENTHUB_CONNECTION_WAIT_TIME = "org.hyperledger.fabric.sdk.eventhub_connection.wait_time";
    public static final String EVENTHUB_RECONNECTION_WARNING_RATE = "org.hyperledger.fabric.sdk.eventhub.reconnection_warning_rate";
    public static final String PEER_EVENT_RECONNECTION_WARNING_RATE = "org.hyperledger.fabric.sdk.peer.reconnection_warning_rate";
    public static final String GENESISBLOCK_WAIT_TIME = "org.hyperledger.fabric.sdk.channel.genesisblock_wait_time";

    ...

    // Default values
    /**
     * Timeout settings
     **/
    defaultProperty(PROPOSAL_WAIT_TIME, "20000");
    defaultProperty(CHANNEL_CONFIG_WAIT_TIME, "15000");
    defaultProperty(ORDERER_RETRY_WAIT_TIME, "200");
    defaultProperty(ORDERER_WAIT_TIME, "10000");
    defaultProperty(PEER_EVENT_REGISTRATION_WAIT_TIME, "5000");
    defaultProperty(PEER_EVENT_RETRY_WAIT_TIME, "500");
    defaultProperty(EVENTHUB_CONNECTION_WAIT_TIME, "5000");
    defaultProperty(GENESISBLOCK_WAIT_TIME, "5000");
    /**
     * This will NOT complete any transaction futures time out and must be kept WELL above any expected future timeout
     * for transactions sent to the Orderer. For internal cleanup only.
     */
    defaultProperty(TRANSACTION_CLEANUP_UP_TIMEOUT_WAIT_TIME, "600000"); //10 min.
```
{:codeblock}

However, you might need to change the default timeout values in your own application. For example, when your application invokes a transaction that needs more than 5000 ms, which is the default timeout value for event hub connection, to response, you might get a failing error because the invoke event ends at 5000 ms before the transaction completes. You can set the system property to overwrite the default values from your client application. Because the default values are initialized before you set the system property, the system property might not take effect. Therefore, you need to set the system property for timeout in a static construct in your client application. See the following example on changing timeout value for event hub connection to 15000 ms in Fabric Java SDK. The file path is `src\main\java\org\hyperledger\fabric\sdk\helper\Config.java`.

```
 public static final String EVENTHUB_CONNECTION_WAIT_TIME = "org.hyperledger.fabric.sdk.eventhub_connection.wait_time";
 private static final long EVENTHUB_CONNECTION_WAIT_TIME_VALUE = 15000;

 static {
     System.setProperty(EVENTHUB_CONNECTION_WAIT_TIME, EVENTHUB_CONNECTION_WAIT_TIME_VALUE);
 }
```
{:codeblock}

If you are using the Node SDK, you can specify the timeout values directly in the method called. As an example, you would use the line below to increase the timeout value for [instantiating a chaincode ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/Channel.html#sendInstantiateProposal "sendInstantiateProposal") to 5 minutes.
```
channel.sendInstantiateProposal(request, 300000);
```
{:codeblock}

## Best practices when using CouchDB
{: #couchdb-indices}

If your ledger data is stored in CouchDB, it is strongly recommended that you create indexes for your CouchDB queries and use them in your chaincode. Indexes allow your applications to retrieve data efficiently as your network adds additional blocks of transactions and entries in the world state. Also, CouchDB allows you to perform rich data queries from your chaincode against data on a channel ledger.

For more information about CouchDB and how to set up indexes, see [CouchDB as the State Database ![External link icon](images/external_link.svg "External link icon")](http://hyperledger-fabric.readthedocs.io/en/release-1.1/couchdb_as_state_database.html "CouchDB as the State Database"){:new_window} in the Hyperledger Fabric documentation. You can also find an example that uses an index with chaincode in the [Fabric CouchDB tutorial ![External link icon](images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/couchdb_tutorial.html).

Avoid using chaincode for queries that will result in a scan of the entire CouchDB database. Full length database scans will result in long response times and will degrade the performance of your network. You can take some of the following steps to avoid long queries:
- Set up indexes with your chaincode.
- If you are issuing rich JSON queries, avoid operators that will result in a full database scan, such as `$or`, `$in` and `$regex`.
- You should use a queryLimit to avoid returning a large set of data that will result in your query timing out. You can then use multiple queries to search the complete database with the limit in place. If you are using a range query, start subsequent queries with the last key returned by the previous query. If you are using rich JSON queries, sort your query using one of the variables in your data. Then use the results to filter your next query using the same variable.
- Do not query the entire database for the purpose of aggregation or reporting. If you want to build a dashboard or collect data as part of your application, you can query an off chain database that replicates the data from your blockchain network. This will allow you to understand the data on the blockchain without degrading the performance of your network or disrupting transactions.

  You can use the Channel Event Hub provided by the Fabric SDK's to build an off chain data store. For example, you can use a block listener to get the latest transactions being added to a channel ledger. The transaction read and write sets can then be used to update a copy of the world state that has been stored in a separate database. For more information, see [How to use the channel-based event service ![External link icon](images/external_link.svg "External link icon")](https://fabric-sdk-node.github.io/tutorial-channel-events.html "How to use the channel-based event service"){:new_window} in the Node SDK Documentation.

## Hosting applications
{: #host-app}

You can host your application on your local file system or push it to {{site.data.keyword.Bluemix_notm}}. To push your application to {{site.data.keyword.Bluemix_notm}}, complete the following steps:
1. Install the [Cloud Foundry Command Line Installer ![External link icon](images/external_link.svg "External link icon")](https://github.com/cloudfoundry/cli/releases).  Test your installation with the `cf` command.
    * If your installation is successful, you should see a bunch of text output in your terminal.
    * If you see "command not found", your installation was either not successful or CF is not added to your system path.
2. Set up API endpoint and log in with your {{site.data.keyword.Bluemix_notm}} ID and password by issuing the following commands:
    ```
    > cf api https://api.ng.bluemix.net
    > cf login
    ```
    {:codeblock}
3. Browse to the directory of your application, and push your application by issuing the following command. This can take several minutes depending on your application size. You can see logs from {{site.data.keyword.Bluemix_notm}} in your terminal. The logs cease when the application is successfully launched.
	```
	> cf push YOUR_APP_NAME_HERE
	```
	{:codeblock}
	You can check your application logs by issuing one of the following commands:
	* `> cf logs YOUR_APP_NAME_HERE`
	* `> cf logs YOUR_APP_NAME_HERE --recent`

## Disconnecting your application from the network
{: #disconnect-app}

Complete the following steps to remove the connection between your application and the blockchain network on {{site.data.keyword.cloud_notm}}.
1. Remove the API endpoint information from your application configuration file. For reference, see [Adding network API endpoints to your application](#api-endpoints).
2. Delete your chaincode container.
  1. In the "Channel" screen of the Network Monitor, locate the channel where your chaincode is installed.
  2. In the specific channel screen, locate the chaincode you want to disable.
  3. Click the **Delete** button, and click **Submit** in the chaincode deletion panel. Your chaincode container will be removed.
    ![Delete a chaincode](images/channel_chaincode.png "Delete a chaincode")
