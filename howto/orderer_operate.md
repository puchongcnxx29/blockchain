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

# Operating an orderer on {{site.data.keyword.cloud_notm}} Private
{: #orderer-operate}

***[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

After you install {{site.data.keyword.blockchainfull}} Platform orderer in ICP, a configmap is created and it contains default settings for environment variables. You can then change or add environment variables for the orderer to configure its behavior.

As a general rule, orderer admins are responsible for bootstrapping and maintaining orderers, but this is not necessary in a SOLO deployment where only one orderer exists. In a SOLO deployment, the orderer admin is responsible for adding new organizations to the orderer system channel.

## Prerequisites

You need to complete a few prerequisite steps from your ICP cluster to operate your orderer.

**Prerequisites:**
- [Configure your CLIs](#orderer-kubectl-configure)
- [Retrieve your orderer endpoint information](#orderer-endpoint)
- [Download your orderer TLS cert](#orderer-tls)

You can then use instructions in this topic to operate your orderer. Note that you will operate your orderer through the command line, which requires to get the `peer` CLI binary. The `peer` CLI binary is downloaded together with the other binaries, such as `configtxlator`. Therefore, many of the commands in this topic start with the word "peer". It's not actually using the peer, but it's just the name of the binary.

### Configuring the CLIs
{: #orderer-kubectl-configure}

You need to use the **kubectl** command line tool to connect to orderer container that runs in ICP.

1. Log in to your ICP cluster UI. Navigate to the **Command Line Tools** tab and click **Cloud Private CLI**. You'll see the following tools that you can download.

   * Install IBM Cloud Private CLI and plug-ins
   * Install Kubectl CLI
   * Install Helm
   * Install Istio CLI

  To operate orderers, you need to use the first **three** tools, among which Helm is optional. Click each of them and run the `curl` commands for the machine type that you're using. Then, issue `chmod` and `sudo mv` commands for each tool. The `chmod` command will change the permission of the CLI in question to make it executable, and the `sudo mv` command will move the file and rename it.

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

2. After you configure **kubectl**, run the following command:

  ```
  kubectl get pods
  ```
  {:codeblock}

  If successful, the command should return a response that is similar to the following example, where `orderer-6cf85f6687-hcxpl` is the name of your orderer container.

  ```
  NAME                                 READY      STATUS             RESTARTS   AGE
  orderer-6cf85f6687-hcxpl          2/2        Running            0          6h
  ```

  You are now ready to use the **kubectl** tool to get the orderer endpoint information.

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

### Retrieving orderer endpoint information
{: #orderer-endpoint}

You need to target your orderer endpoint to make updates to the orderer system channel. You will need to be a [Cluster administrator ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/user_management/assign_role.html "Cluster administrator roles and actions") to complete the following steps:

1. Log in to your ICP console and click the **Menu** icon in the upper left corner.
2. Click **Workload** > **Helm Releases**.
3. Find the name of your Helm Release and open the Helm Release details panel.
4. Scroll down to the **Notes** section at the bottom of the panel. The **Notes** section includes a set of commands to help you operate your orderer deployment.
5. If you have not already, follow the instructions to configure the [kubeclt CLI](#orderer-kubectl-configure). You need to use it to interact with your orderer container.
6. In your CLI, run the first command in the note, which follows **1. Get the application URL by running these commands:** This command will print out the orderer URL and port, which is similar to following example. Save this URL and you need to use it in future commands to set your Proxy address and external node port.

  ```
  http://9.30.94.174:30159
  ```

In this example, the Proxy IP address is `9.30.94.174` and the external node port which corresponds to 7050 is `30159`.  

**Note:** If you are deploying your orderer behind a firewall, you need to open a passthru to enable the network on the platform to complete this task.

### Downloading your orderer TLS cert
{: #orderer-tls}

You need to download your orderer TLS certificate and pass it to your commands to communicate with your orderer from a remote client. You will need to be a [Cluster administrator ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/user_management/assign_role.html "Cluster administrator roles and actions") to complete the following steps:

1. Log in to your ICP console and click the **Menu** icon in the upper left corner.
2. Click **Workload** > **Helm Releases**.
3. Find the name of your Helm Release and open the Helm Release details panel.
4. Scroll down to the **Notes** section at the bottom of the panel. The **Notes** section includes a set of commands to help you operate your orderer deployment.
5. If you have not already, follow the instructions to configure the [kubeclt CLI](#ca-kubectl-configure). You need to use it to interact with your orderer container.
6. In your CLI, run the third command in the note, which follows **3. Get orderer TLS root cert file**.  This command will save your TLS certificate as the file `cert.pem` on your local machine.

  **Note:** Before you run the commands, you can remove `app=orderer` as shown in the command below:

  ```
  export POD_NAME=$(kubectl get pods --namespace blockchain-dev -l "release=pa-orderer3" -o jsonpath="{.items[0].metadata.name}")
  ```
  {:codeblock}

7. Move the generated certificate to a location where you can reference it in future commands, and rename it to `orderertls.pem`.

  ```
  mkdir $HOME/fabric-ca-client/orderer-tls
  cp cert.pem $HOME/fabric-ca-client/orderer-tls/orderertls.pem
  ```
  {:codeblock}

### Managing the certificates on your local system
{: #manage-certs}

Switch to the directory where the orderer admin MSP folder is generated. Depending how you followed example steps in this documentation, or how many components you are deploying, you can find the MSP folder in `$HOME/fabric-ca-client/orderer-admin/msp` or `$HOME/fabric-ca-client/peer-admin/msp`

Before you can operate the orderer, you need to do some management of the certificates on your local machine. You also need to ensure that you can access the TLS certificates from the orderer. For more information about the certificates to use, see [Membership Service Providers](CA_operate.html#msp) in [Operating a Certificate Authority on {{site.data.keyword.cloud_notm}} Private](CA_operate.html).

1. Move your orderer admin's signCert to a new folder that is named `admincerts`:

  ```
  cd $HOME/fabric-ca-client/orderer-admin/msp
  mkdir admincerts
  cp signcerts/cert.pem admincerts/cert.pem
  ```
  {:codeblock}

2. Ensure that you [download your orderer TLS certificate](#orderer-tls) and can reference it from your command line. If you followed the example commands in this documentation, you can find this TLS cert in the `$HOME/fabric-ca-client/orderer-tls/orderertls.pem` file.

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

## Adding organizations to the orderer system channel
{: #add-organizations-to-consortium}

**Note:** If you plan to deploy a peer and connect it to a Starter or Enterprise plan network, then you can skip this step.

When you deploy an orderer by using {{site.data.keyword.blockchainfull_notm}} Platform for ICP Helm chart, the genesis block is automatically created with the orderer organization as the only administrator of the system channel. This means that the orderer organization admin has the sole ability to add organizations to the consortium. The policy that governs this, which is known as the `mod_policy` for the orderer system channel, should not be changed after new organizations are added to the orderer system channel.

Adding organizations to the orderer system channel makes them part of the "consortium", which is the list of members who can, by default, create channels. Being a member of the consortium also makes it easy to be added to a channel by using the certificates and information that is listed in the system channel.

Updating the orderer system channel is achieved through "configuration update transactions", in which the relevant MSP information of the organizations is added to the channel configuration of the orderer system channel.

Adding organizations to the orderer system channel is essentially the same flow as updating any channel's configuration to add an organization. However, you need to make a few changes because the channel to update is not an application channel and the relevant admin is the orderer admin instead of a peer organization.

Note that you can add an organization to a channel without joining the system channel first. For more information, see the [Adding an Org to a Channel Tutorial](https://hyperledger-fabric.readthedocs.io/en/release-1.2/channel_update_tutorial.html) in the Hyperledger Fabric documentation.

The following list shows the general steps and the tasks will be performed by different sets of organizations of your consortium.

1. Each organization to join the consortium needs to [prepare an organization definition](peer_operate_icp.html#organization-definition).
2. The orderer organization admin [forms the consortium](#consortium) by adding organizations to the orderer system channel.
3. Any organization of the consortium can [create a new channel](peer_operate_icp.html#peer-icp-channeltx) by preparing a channel configuration transaction.

## Getting the Fabric tools
{: #get-fabric-tools}

You need to download the following Hyperledger Fabric tools to update the system channel.
- [peer](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/peercommand.html), which will allow you to fetch the genesis block and update the system channel.
- [configtxlator](https://hyperledger-fabric.readthedocs.io/en/release-1.2/commands/configtxlator.html), which translates the protobuf format of a channel configuration into the JSON format that is more easily to read and update.

1. Decide where you want to store the tools and then run this command:

  ```
  curl -sSL http://bit.ly/2ysbOFE | bash -s 1.2.1 1.2.1 -d -s
  ```
  {:codeblock}

2. Set your path to your tools, where the above Fabric tools are downloaded:

  ```
  export PATH=$PATH:<full_path_to_bin_folder>
  ```
  {:codeblock}

  For example, if you installed the binaries in your home directory you would set your `PATH` as:

  ```
  export PATH=$PATH:$HOME/bin
  ```
  {:codeblock}

3. Downloading the binaries will create a config folder containing a core.yaml, orderer.yaml and a configtx.yaml file. Set the Fabric config path to this config directory:

  ```
  export FABRIC_CFG_PATH=<full_path_to_config_folder>
  ```
  {:codeblock}

  For example:
  ```
  export FABRIC_CFG_PATH=$HOME/config
  ```
  {:codeblock}

## Creating an organization definition
{: #org-definition}

The **definition** of an organization contains the organization name (MSP ID) and the relevant certificates. The system channel and application channels will use this definition to include your organization in the policies that control channel creation, updates, and transaction endorsement. Each organization that wants to join the consortium need to complete this step. To learn more, see [preparing an organization definition](peer_operate_icp.html#organization-definition).

## Forming the Consortium
{: #consortium}

Recall the high-level flow for forming a consortium:

1. The orderer system channel is formed with only the orderer organization as a member of the system channel. That organization can make updates without requiring additional signatures.
2. The orderer organization admin receives organization definitions from members that want to join the consortium. Orderer organization uses the organization definitions to update the configuration of the orderer system channel.

### Getting the organization definitions

The orderer needs to receive the [organization definitions](peer_operate_icp.html#organization-definition) from members who want to join the consortium. This needs to be completed in an out of band operation with other members sending you the JSON files that include their MSP ID and crypto material. For reference in the commands below, we assume that you have created a folder that is named `org-definitions` and placed all of the relevant files in that directory.

### Fetching the genesis block of the system channel

1. Set the following environment variables by using your orderer org MSP name that is established during deployment, the path to your orderer MSP, and the path to the TLSCA cert of your orderer.

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

  Replace the fields with your own information.

    - Replace `<CORE_PEER_LOCALMSPID>` with the MSP ID of your orderer organization. It is also visible inside the orderer container by running the following commands, replacing `<orderer pod name>` with the value of your orderer's pod:

      ```
      kubectl exec -it <orderer pod name> -c orderer sh
      $ env | grep ORDERER_GENERAL_LOCALMSPID
      ```
      {:codeblock}

      The output might look similar to:

      ```
      ORDERER_GENERAL_LOCALMSPID=ordererOrg
      ```

      Therefore, the `CORE_PEER_LOCALMSPID` is ordererOrg.

    - Replace `<CORE_PEER_MSPCONFIGPATH>` with the path to the admin MSP folder of the orderer organization.
    - Replace `<CORE_PEER_TLS_ROOTCERT_FILE>` with the path to the TLS CA cert.
    - Replace `<PROXY_IP>` with the proxy ip address from [orderer endpoint information](#orderer-endpoint)
    - Replace `<EXTERNAL_NODE_PORT>` with the external node port from [orderer endpoint information](#orderer-endpoint)

  Your environment variables might look like the following example:

  ```
  export FABRIC_CFG_PATH=$HOME/config
  export CORE_PEER_LOCALMSPID=ordererOrg
  export CORE_PEER_MSPCONFIGPATH=$HOME/fabric-ca-client/orderer-admin/msp/
  export CORE_PEER_TLS_ROOTCERT_FILE=$HOME/fabric-ca-client/orderer-tls/orderertls.pem 
  export CORE_PEER_TLS_ENABLED=true
  export PROXY = 9.30.94.174
  export ORDERER_PORT = 30159
  ```

  You can check these environment variables at any point by issuing the following commands:

  ```
  env | grep CORE
  env | grep PATH
  ```
  {:codeblock}

2. Set the channel name as an environment variable. The name for the orderer system channel will always be `test-system-channel-name`.

  ```
  export CHANNEL_NAME=test-system-channel-name
  ```
  {:codeblock}

3. Fetch the genesis block of the system channel. First, create a directory called `configupdate` inside `org-definitions` to store config files generated during the config update process. You can call this directory anything you want, but the following sample commands will use `configupdate.`

  ```
  peer channel fetch config ./configupdate/genesis.pb -o $PROXY:$ORDERER_PORT -c $CHANNEL_NAME --tls --cafile <PATH_TO_ORDERER_TLS_CERT>
  ```
  {:codeblock}

  Replace `<orderer_TLS_root_cert_file>` with the path to the `orderertls.pem` file that you created in this [step](#orderer-tls). For example, your command might look like the following example:

  ```
  peer channel fetch config ./configupdate/genesis.pb -o $PROXY:$ORDERER_PORT -c $CHANNEL_NAME --tls  --cafile $HOME/fabric-ca-client/orderer-tls/orderertls.pem
  ```
  {:codeblock}

  If successful, this command will generate the following output.

  ```
  2018-10-25 20:53:06.377 UTC [cli/common] readBlock -> INFO 002 Received block: 0
  2018-10-25 20:53:06.380 UTC [cli/common] readBlock -> INFO 003 Received block: 0
  ```

  You can also find the `genesis.pb` block in your file system afterwards.

### Creating the system channel update
{: #system-channel-update}

The downloaded [Fabric tool](#get-fabric-tools) `configtxtlator` translates the protobuf format of a channel configuration to the JSON format, and vice vera.

These steps follow the general flow of the channel update tutorial about [converting the block into JSON format]( https://hyperledger-fabric.readthedocs.io/en/release-1.2/channel_update_tutorial.html#convert-the-configuration-to-json-and-trim-it-down). You need to make some changes to the commands in the tutorial to reflect the fact that you are updating the orderer system channel rather than an application channel. You can visit the tutorial for more detail on this process. This section simply provides the commands for you.

1. Copy the organization definition JSON file from the folder where you [created your organization](peer_operate_icp.html#organization-definition) to your `configupdate` folder. In the example command below, the organization definition JSON file is `org1definition.json`:

   ```
   cp <path_to_config_folder>/org1definition.json $HOME/fabric-ca-client/org-definitions/configupdate
   ```
   {:codeblock}

2. Convert the genesis block into JSON format. Run the following commands:

  ```
  cd configupdate
  configtxlator proto_decode --input genesis.pb --type common.Block --output ./config_block.json
  jq .data.data[0].payload.data.config ./config_block.json  > config.json
  ```
  {:codeblock}

3. Run the following command to add the crypto material of an organization to the consortium configuration. Replace <NEWORGMSP> with the organization MSP ID for the [organization that you created](peer_operate_icp.html#organization-definition).

  ```
  jq -s '.[0] * {"channel_group":{"groups":{"Consortiums":{"groups":{"SampleConsortium":{"groups": {"<NEWORGMSP>":.[1]}}}}}}}' config.json ./orgdefinition.json > modified_config.json
  ```
  {:codeblock}

  The command might look like the following example:

  ```
  jq -s '.[0] * {"channel_group":{"groups":{"Consortiums":{"groups":{"SampleConsortium":{"groups": {"org1":.[1]}}}}}}}' config.json ./orgdefinition.json > modified_config.json
  ```

4. Repeat this command for each organization that needs to join the consortium. Ensure that you change the `./orgdefinition.json` file to be the new organization JSON file.

  ```
  jq -s '.[0] * {"channel_group":{"groups":{"Consortiums":{"groups":{"SampleConsortium":{"groups": {"org1":.[1]}}}}}}}' config.json ./orgdefinition.json > modified_config.json
  ```
  {:codeblock}

5. Run the following commands to convert the `modified_config.json` file to the configuration update in JSON format.

  ```
  configtxlator proto_encode --input config.json --type common.Config --output config.pb 
  configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb 
  configtxlator compute_update --channel_id $CHANNEL_NAME --original config.pb --updated modified_config.pb --output config_update.pb 
  configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate --output config_update.json
  ```
  {:codeblock}

6. After we complete these commands, put an envelope on the `config_update`.

  ```
  echo '{"payload":{"header":{"channel_header":{"channel_id":"'$CHANNEL_NAME'", "type":2}},"data":{"config_update":'$(cat config_update.json)'}}}' | jq . > config_update_in_envelope.json
  ```
  {:codeblock}

7. Run the following command to convert the configuration update back to the protobuf format:

  ```
  configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope --output ./config_update_in_envelope.pb
  ```
  {:codeblock}

### Sending the update to the system channel

Before you complete these steps, ensure that your `FABRIC_CFG_PATH`, `$PROXY`  and `ORDERER_PORT`  is set correctly. Run the following command:

```
export ORDERER_CA=<path and file name of the orderer TLS CA cert>
export PROXY = <proxy ip address from [orderer endpoint information](#orderer-endpoint)>
export ORDERER_PORT = <external node port from [orderer endpoint information](#orderer-endpoint)>
```
{:codeblock}

Your environment variables should be similar to the following example:

```
ORDERER_CA = /Users/chandra/fabric-ca-client/orderer-tls/orderertls.pem
CORE_PEER_TLS_ROOTCERT_FILE=/Users/chandra/fabric-ca-client/orderer-tls/orderertls.pem
CORE_PEER_LOCALMSPID=ordererOrg
CORE_PEER_TLS_ENABLED=true
CORE_PEER_MSPCONFIGPATH=/Users/chandra/fabric-ca-client/orderer-admin/msp
PROXY = 9.30.94.174
ORDERER_PORT = 30159

```

After you create the channel configuration update, which is `config_update_in_envelope.json`, the orderer organization can submit the channel update by using the following command:

```
peer channel update -f config_update_in_envelope.pb -c $CHANNEL_NAME -o $PROXY:$ORDERER_PORT --tls --cafile $ORDERER_CA
```
{:codeblock}

This command simultaneously signs the update request and sends it to the orderer. Because the orderer organization is the only administrator of the system channel, only one signature is required for this update to be a valid request. If the system channel update is successful, you will see the following output:

```
2018-10-28 23:44:00.498 UTC [channelCmd] update -> INFO 002 Successfully submitted channel update
```

It is possible to include multiple organization definitions in a single orderer system channel configuration update, but it's a best practice to update the channel with one organization at a time and check to make sure that the update was successful.

## Viewing the orderer logs
{: #orderer-view-logs}

Component logs can be viewed from the command line by using the [`kubectl CLI commands`](#ca-kubectl-configure) or through [Kibana ![External link icon](../images/external_link.svg "External link icon")](https://www.elastic.co/products/kibana "Your window into the Elastic Search"), which is included in your ICP cluster.

- Use the `kubectl logs` command to view the container logs inside the pod. If you are unsure of your pod name, run the following command to view your list of pods.

  ```
  kubectl get pods
  ```
  {:codeblock}

  Then, run the following command to retrieve the logs for the orderer container that resides inside the pod by replacing `<pod_name>` with the name of your pod from the command output above:

  ```
  kubectl logs <pod_name> -c orderer
  ```
  {:codeblock}

  For more information about the `kubectl logs` command, see [Kubernetes documentation ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs “Getting Started”)

- Alternatively, you can access logs by using the  [ICP cluster management console](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/troubleshoot/events.html), which opens the logs in Kibana.

  **Note:** When you view your logs in Kibana, you might receive the response `No results found`. This condition can occur if ICP uses your worker node IP address as its hostname. To resolve this problem, remove the filter that begins with `node.hostname.keyword` at the top of the panel and the logs will become visible.

## Troubleshooting
{: #orderer-troubleshooting}

### **Problem:** `peer channel update` command fails with an error.
{: #orderer-peer-channel-update-error}

It is possible to receive the following error when you run a `peer channel update` command:

```
INFO 001 Endorser and orderer connections initialized Error: got unexpected status: BAD_REQUEST -- initializing channelconfig failed: could not create channel Consortiums sub-group config: Attempted to define two different versions of MSP: Org1
```

This error can be caused when the peer is using the same organization MSP ID as the orderer.  

**Solution:**  

To resolve this problem you need to redeploy your peer specifying an Organization MSP ID that is different then what was used when the orderer was deployed.
