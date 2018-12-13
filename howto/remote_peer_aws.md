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

# Deploying {{site.data.keyword.blockchainfull_notm}} Platform for AWS
{: #remote-peer-aws}


***[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


These instructions describe how to use an Amazon Web Services (AWS) Quick Start template to create an {{site.data.keyword.blockchainfull}}  Platform for AWS peer and then connect it to a network on an {{site.data.keyword.blockchainfull_notm}} Platform.
{:shortdesc}

For more information about AWS, see the [AWS overview document ![External link icon](../images/external_link.svg "External link icon")](https://d1.awsstatic.com/whitepapers/aws-overview.pdf "AWS overview document").

Before you deploy {{site.data.keyword.blockchainfull_notm}} Platform for AWS peers, review the [peer considerations](remote_peer.html#remote-peer-limitations).

## Prerequisites
{: #prerequisites-aws}

To use a {{site.data.keyword.blockchainfull_notm}} Platform for AWS peer (peer), you must have an organization that is a member of a blockchain network that is hosted on IBM Blockchain Platform. You need to use the Network Monitor on IBM Cloud to access network credentials and API endpoints of your network. If you are not a member of any blockchain network, you need to create or join a network. For more information, see [Creating a network](../get_start.html#creating-a-network) or [Joining a network](../get_start.html#joining-a-network).

The default VPC instance type for the peer is `m4.xlarge`.  You should optimize the instance type you choose based on your cpu, memory and storage requirements. The peer requires at least:  
-	2x CPU
-	2 GB RAM
-	4 GB space for chaincode
-	10 GB space for the ledger with ability to grow as the ledger expands.

These minimum resource levels are sufficient for testing and experimentation. For a production environment it is important to allocate a sufficiently large amount of storage, 100GB for example. The amount of storage used will depend on the number of transactions and the number of signatures required from your network. If you exhaust the storage on your peer, you must <!-- either expand the storage or --> deploy a new peer with a larger file system and let it sync via your other peer(s) on the same channel(s).


## Deployment options
{: #remote-peer-aws-deploy-options}

The Quick Start provides two deployment options:

* Deploy the {{site.data.keyword.blockchainfull_notm}} Platform for AWS into a new VPC (end-to-end deployment). This option builds a new AWS environment consisting of the VPC, subnets, NAT gateways, security groups, bastion hosts, and other infrastructure components, and then deploys the peer into this new VPC.

* Deploy the {{site.data.keyword.blockchainfull_notm}} Platform for AWS into an existing VPC. This option provisions the {{site.data.keyword.blockchainfull_notm}} Platform for AWS peer in your existing AWS infrastructure. There are separate templates for these options where you can configure CIDR blocks, instance types, and  Peer settings, as discussed later in this guide.

## Step One: Prepare your AWS account
{: #remote-peer-aws-account}

1. If you don’t already have an AWS account, create one [here ![External link icon](../images/external_link.svg "External link icon")](https://aws.amazon.com "https//aws/amazon.com") by following the on-screen instructions.

2. Use the region selector in the navigation bar to choose the AWS Region where you want to deploy the  peer in AWS.

3. Create a key pair in your preferred region.

4. If necessary, request a service limit increase for the Amazon EC2 <type> instance type. You might need to do this if you already have an existing deployment that uses this instance type, and you think you might exceed the default limit with this deployment.

## Step Two: Retrieve your remote peer configuration information
{: #aws-network-endpoints}

You need to provide the API endpoints of your network to your  peer during configuration. These endpoints allow a  peer to find and connect to the network on {{site.data.keyword.blockchainfull_notm}} Platform. On the **Overview** screen of your Network Monitor, click the **Remote Peer Configuration** button.

![Remote Peer Configuration](../images/myresources_starter.png "Remote Peer Configuration")
*Figure 1. Remote Peer Configuration Panel*

A pop-up window opens and displays the values of the following fields. Save the values of the following fields, they are required when you configure the  peer using the AWS Quick Start template.

- **Organization MSP**
- **Certificate Authority (CA) Name**
- **Certificate Authority (CA) URL**
- **Certificate Authority (CA) TLS Certificate**

You can copy and paste each field directly into the Quick Start template, or save them to a JSON file by clicking the **Download** link.

**Note:** If you download the information in JSON, you need to convert the TLS certificate into PEM format before you provide it to the  peer. Convert the **Certificate Authority (CA) TLS Certificate** in the JSON file you downloaded into PEM format by issuing the command:
```
echo -e "<CERT>" > admin.pem
```
{:codeblock}

Replace `<CERT>` with the value of the **Certificate Authority (CA) TLS Certificate**. Then, when prompted for the **Certificate Authority (CA) TLS Certificate** in the Quick Start template, `cat` the admin.pem file and then copy and paste the contents into the field.  

## Step Three: Register a {{site.data.keyword.blockchainfull_notm}} Platform for AWS peer
{: #aws-register-peer}

You need to add a new peer identity to your network on the {{site.data.keyword.blockchainfull_notm}} Platform before the {{site.data.keyword.blockchainfull_notm}} Platform for AWS peer can join the network. Complete the following steps to register a peer.

**Note:** For High Availability, the Quick Start template creates two peer nodes across two availability zones. Therefore, two enroll ids and secrets are required. **Repeat this process twice to generate two enroll ids and secrets.**

1. Log in to the Network Monitor of your network on {{site.data.keyword.blockchainfull_notm}} Platform. On the "Certificate Authority" screen of your Network Monitor, you can view all the identities that have been registered with the network, such as your admin or client applications.
  ![CA screen](../images/CA_screen_starter.png "CA screen")
  *Figure 2. CA screen*

2. Click the **Add User** button on the panel. A pop-up screen opens and allows you to register your peer to the network after filling out the fields below. **Save the value of the ID and Secret for later when you configure your peer in the Quick Start template.**
  - **Enroll ID:** The name you want to use for your peer, which is referred to as your `enroll ID` when you configure your peer. **Save this value** for future usage.
  - **Enroll Secret:** The password you want to use for your peer, which is referred to as your `enroll Secret` when you configure your peer. **Save this Value** for future usage.
  - **Type:** Select `peer` for this field.
  - **Affiliation:** This is the affiliation under your organization, `org1` for example, that your  peer belongs to. You can specify a new affiliation or use an existing one.
  - **Maximum Enrollments:** You can use this field to limit the number of times your can enroll or generate certificates using this identity. If not specified, the value defaults to unlimited enrollments.

  After you fill in the fields, click **Submit** to register the peer. The registered peer is then listed in the table as an identity on the network. As a security measure, use each identity, and the accompanying enrollID and secret, to deploy only one peer. Do not reuse peer ID's and passwords.

## Step Four: Launch the Quick Start
{: #remote-peer-aws-launchqs}

You are responsible for the cost of the AWS services that you use while you run this Quick Start reference deployment. There is no additional cost for using this Quick Start. For full details, see the pricing pages for each AWS service that you need to use in this Quick Start. Prices are subject to change.

1. Choose one of the following options to launch the AWS CloudFormation template into your AWS account. For help choosing an option, see deployment options earlier in this guide. Each deployment takes about 10 minutes to complete.  

  * [Deploy {{site.data.keyword.blockchainfull_notm}} Platform for AWS into a new VPC on AWS ![External link icon](../images/external_link.svg "External link icon")](https://fwd.aws/v43nk "Deploy {{site.data.keyword.blockchainfull_notm}} Platform for AWS into a new VPC on AWS").  

  * [Deploy {{site.data.keyword.blockchainfull_notm}} Platform for AWS into an existing VPC on AWS ![External link icon](../images/external_link.svg "External link icon")](https://fwd.aws/zrP4g "Deploy {{site.data.keyword.blockchainfull_notm}} Platform for AWS into an existing VPC on AWS").

  **Important:**     
  If you’re deploying the {{site.data.keyword.blockchainfull_notm}} Platform for AWS into an existing VPC, make sure that your VPC has two public subnets in different Availability Zones for the database instances. These subnets require NAT gateways or NAT instances in their route tables, to allow the instances to download packages and software without exposing them to the internet. You will also need the domain name option configured in the DHCP options as explained in the Amazon VPC documentation.  

  Also, be sure to create a security group tied to your existing VPC and add inbound rules on ports 22 and 7051 to this security group. TCP connections on port 22 allow for SSH access to the generated instance while TCP connections on port 7051 allow for external gRPC access to the peer instance (needed for operating the peer using the Fabric tools CLI and Fabric SDKs). You will be prompted for these VPC settings when you launch the Quick Start.

2. Check the region that is displayed in the upper-right corner of the navigation bar, and change it if necessary. This is where the network infrastructure for the peer will be built. The template is launched in the US East (Ohio) Region by default.

3. On the Select Template page, keep the default setting for the template URL, and then choose `Next`.

4. On the Specify Details page, change the stack name if needed. Review the parameters for the template. Provide values for the parameters that require input. For all other parameters, review the default settings and customize them as necessary. When you finish reviewing and customizing the parameters, choose `Next`.

In the following tables, parameters are listed by category and described separately for the two deployment options:

  * [Parameters for deploying {{site.data.keyword.blockchainfull_notm}} Platform for AWS into a new VPC](#remote-peer-aws-parameters-newvpc)

  * [Parameters for deploying {{site.data.keyword.blockchainfull_notm}} Platform for AWS into an existing VPC](#remote-peer-aws-parameters-existvpc).

### Parameters for deploying {{site.data.keyword.blockchainfull_notm}} Platform for AWS  into a new VPC
{: #remote-peer-aws-parameters-newvpc}

The following table lists the configurable parameters of the AWS chart and their default values. All values are required.

|  Parameter    | Description | Default |
| --------------|-------------|---------|
| `Stack name` |The stack name is an identifier that helps you find a particular stack from a list of stacks. A stack name can contain only alphanumeric characters (case-sensitive) and hyphens. It must start with an alphabetic character and can't be longer than 128 characters.| |
| | | |
| **Network Configuration** | |
| `Availability Zones` |The two Availability Zones to use for the subnets in the VPC. Note: The logical order is preserved. | |
| `Allowed SSH access CIDR` | Allowed [CIDR block ![External link icon](../images/external_link.svg "External link icon")](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html#vpc-resize "VPCs and Subnets") for external SSH access to the IBM Blockchain Peer instances. It can be set to 0.0.0.0/0 to allow access from anywhere (not recommended). | |
| `PeerEndpointAccessCIDR` | Allowed [CIDR ![External link icon](../images/external_link.svg "External link icon")](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html#vpc-resize "VPCs and Subnets") block for external gRPC access to the IBM Blockchain Peer instances. Typically set to 0.0.0.0/0 to allow access from anywhere (not recommended). | |
| | | |
| **Amazon EC2 Configuration** | | |
| `InstanceType` | Type of EC2 instance for the peer instances. | m4.xlarge |
| `KeyPairName` | Name of an existing EC2 key pair within the AWS region. You must generate this. | |
| | | |
|** IBM Blockchian Configuration** | |
| `IBMBlockchainVersion` | IBM Blockchain version to deploy. | 1.2.1 |
| `StateDatabase` | The type of database to use for storing blockchain state. This selection should match the State Database type used by the rest of the network. | CouchDB|
| `PeerVolumeSize` | Size of the EBS Volume Size used to store persistent data (ledger, state database, MSP) for the peer in GBs. | 100 |
| `Peer 1 enroll ID`| The Enroll ID you entered in your IBM Blockchain Platform UI Certificate Authority panel for your first peer. |  |
| `Peer 1 enroll secret` | The Enroll Secret you entered in your IBM Blockchain Platform UI Certificate Authority panel for your first peer. | |
| `Peer 2 enroll ID` | The Enroll ID you entered in your IBM Blockchain Platform UI Certificate Authority panel for your second peer. | |
| `Peer 2 enroll secret` | The Enroll Secret you entered in your IBM Blockchain Platform UI Certificate Authority panel for your second peer. | |
| | | |
|**IBM Blockchain Service Credentials**| | |
| `Organization MSP` | This value can be found in your IBM Blockchain Platform UI. Click the Remote Peer Configuration button on the Overview panel and copy and paste that information here. | |
| `Certificate Authority (CA) Name` | This value can be found in your IBM Blockchain Platform UI. Click the Remote Peer Configuration button on the Overview panel and  copy and paste that information here.| |
| `Certificate Authority (CA) URL` | This value can be found in your IBM Blockchain Platform UI. Click the Remote Peer Configuration button on the Overview panel and  copy and paste that information here, including the port. If not specified the default port is 443. | |
| `Certificate Authority (CA)  TLS Certificate`| This value can be found in your IBM Blockchain Platform UI. Click the Remote Peer Configuration button on the Overview panel and  copy and paste that information here.| |
| | | |
|**Other Parameters**| | |
| `QSS3BucketName` | S3 bucket name for the Quick Start assets. Quick Start bucket name can include numbers, lowercase letters, uppercase letters, and hyphens (-). It cannot start or end with a hyphen (-). | `aws-quickstart` |
| `QSS3KeyPrefix` | S3 key prefix for the Quick Start assets. Quick Start key prefix can include numbers, lowercase letters, uppercase letters, hyphens (-), and forward slash (/). | `quickstart-ibm-fabric/` |

1. On the Options page, you can specify tags (key-value pairs) for resources in your stack and set advanced options. When you’re done, choose Next.

2. On the Review page, review and confirm the template settings. Under Capabilities, select the check box to acknowledge that the template will create IAM resources.

3. Choose Create to deploy the stack.

4. Monitor the status of the stack. When the status of all the stacks is `CREATE_COMPLETE`, the Peer cluster is ready. After successful completion you should have a root stack with four nested stacks if couchDB was chosen or two nested stacks if levelDB was chosen.

5. Use the information displayed in the Outputs tab for the stack to view the resources that were created.


### Parameters for deploying an {{site.data.keyword.blockchainfull_notm}} Platform Peer into an existing VPC
{: #remote-peer-aws-parameters-existvpc}

If you are deploying the {{site.data.keyword.blockchainfull_notm}} Platform for AWS peer into an existing VPC, the following considerations should be taken:

 - Ensure that your VPC has two private subnets in different Availability Zones for the database instances. These subnets require NAT gateways or NAT instances in their route tables, to allow the instances to download packages and software without exposing them to the internet.

 - Configure the domain name option in the DHCP options as explained in the [Amazon VPC documentation ![External link icon](../images/external_link.svg "External link icon")](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_DHCP_Options.html "DHCP Options Sets").  

- Create a security group tied to your existing VPC and add inbound rules on ports 22 and 7051 to this security group. TCP connections on port 22 allow for SSH access to the generated instance while TCP connections on port 7051 allow for external gRPC access to the peer instance (needed for operating the peer using the Fabric tools CLI and Fabric SDKs). You will be prompted for these VPC settings when you launch the Quick Start.

 When deploying a {{site.data.keyword.blockchainfull_notm}} Platform for AWS peer into an existing VPC, the following parameters replace the parameters in the corresponding sections [above](#remote-peer-aws-parameters-newvpc):

|  Parameter    | Description | Default |
| --------------|-------------|---------|
| **Network Configuration** | | |
| `VPCID` |	ID of your existing VPC for deployment.| |
| `AvailabilityZone1` | The Availability Zone in which to deploy the first peer node. | |
| `SubnetID1` |	ID of the subnet to use for the first peer node. The subnet should belong to the chosen VPC. To find the subnets belonging to the VPC, please check your AWS VPC dashboard, and select "subnets" menu.| |
| `AvailabilityZone2` | The Availability Zone in which to deploy the second peer node. | |
| `SubnetID2` |	ID of the subnet to use for the second peer node. The subnet should belong to the chosen VPC. To find the subnets belonging to the VPC, please check your AWS VPC dashboard, and select "subnets" menu. | |
| | | |
| **Amazon EC2 Configuration**| | |
| `InstanceType` 	| Type of EC2 instance for the peer instances. 	| m4.xlarge |
| `KeyPairName` |	Name of an existing EC2 key pair within the AWS region. You must generate this. | |
| `SecurityGroup` | ID of an existing EC2 security group within the AWS region. You should allow inbound TCP connections on ports 22 and 7051. |	| |

## Step Five: Test the Deployment
{: #remote-peer-aws-test}

When the AWS CloudFormation template has successfully created the stack, two {{site.data.keyword.blockchainfull_notm}} Platform for AWS peer instances will be running in your AWS account. The instances will be named based on the combination of the `Organization MSP` and the `Peer enroll id` that is specified in the Quick Start template. For example, `org1-remotepeer1`.  

![Peer AWS EC2 Instances](../images/remote_peer_AWS_EC2_instances.png "Peer AWS EC2 Instances")  
*Figure 3. Remote Peer on AWS EC2 instances*

To verify the peer is running:

  * SSH into your newly created VPC by selecting the instance in the AWS console (click **Services > EC2 > Instances**) and then clicking the **Connect** button. Follow the instructions from AWS for issuing the `ssh` command.  
  * From the command line run `docker ps` to view the running containers. Each virtual machine includes a peer container and an ecs-agent. If you selected CouchDB as your ledger database, there will also be a CouchDB container as well.

  ```
  CONTAINER ID        IMAGE                                STATUS              PORTS                          NAMES
  fb3c49fe52fe        amazon/amazon-ecs-agent:latest       Created                                            ecs-agent
  667780cf3cd3        ibmblockchain/fabric-peer:1.2.1      Up                  0.0.0.0:7051->7051/tcp         peer
  2aa143c81027        ibmblockchain/fabric-couchdb:0.4.6   Up                  4369/tcp, 5984/tcp, 9100/tcp   couchdb
  ```

  * You can create a shell session inside the peer container by running `docker exec -it peer sh`.

Additionally, to verify that the  peer connection is working to your {{site.data.keyword.blockchainfull_notm}} Platform network, you can run a peer CLI command from inside the  peer container. Run the `peer channel fetch` CLI command to fetch the genesis block from the channel:

1. Retrieve configuration information from your `Connection Profile` available in the Overview panel of your Network Monitor. Click **Connection Profile** and then **Download**.

   - Find the orderer URL by searching for **orderers**, which is located under `orderers > url`. Make a note of the URL that begins with the network name. The URL is similar to the following example:

   ```
   ash-zbc07b.4.secure.blockchain.ibm.com:21239
   ```

   - Find the name of your organization by searching for **organizations**. This should be the same organization you use to register your  peer. You can find your organization's name together with its associated `mspid`. This value is also availabe in the Network Monitor Overview panel. Click on the **Remote Peer Configuration** button. The value is displayed under `Organization MSP`. Make a note of the value of the `mspid`.

   - If you have not already done so, create a shell session inside the peer container by running `docker exec -it peer sh`.

   ```
   docker exec -it peer sh
   ```
   {:codeblock}

   - Copy the orderers TLS certificate from the connections profile to the  peer. Navigate to the **orderers** section. Copy the certificate that follows "pem:", beginning with -----BEGIN CERTIFICATE----- and ending with -----END CERTIFICATE-----. Do not include the quotation marks. Run the following command from the command line, replacing `<orderer cert>` with the content you copied from the creds.json file.

   ```
   echo -e "<orderer cert>" > /etc/hyperledger/<PEER_ENROLL_ID>/orderer_tlscacert.pem
   ```
   {:codeblock}

   Replace  `<PEER_ENROLL_ID>` with the enroll ID that is specified in the Quick Start template and is associated with this peer instance.

2. Your organization needs to be added to a channel in the network before you can fetch the genesis block.

  - You can start a new channel for the  peer. As the channel initiator, you can automatically include your organization during [channel creation](create_channel.html#creating-a-channel).

  - Another member of the blockchain network can also add your organization to an existing channel using a [channel update](create_channel.html#updating-a-channel).

  - After your organization is added to a channel, you need to add your peer's signing certificate to the channel. The  peer uploads its signing cert during installation, so that you need to only synchronize the certificate to the channel. On the "Channels" screen of the Network Monitor, locate the channel that your organization joined and select **Sync Certificate** from the drop-down list under the **Action** header. This action synchronizes the certificates across all the peers on the channel.

3. Run the following commands to set the environment variables in the peer container.

   ```
   export ORDERER_1=<ORDERER_URL>
   export CHANNEL=<CHANNEL_NAME>
   export ORGID=<ORGANIZATION_MSP_ID>
   export PEERADDR=<PEER_ADDR>
   ```
   {:codeblock}

   Replace the fields with your own information.
     - Replace `<ORDERER_URL>` with the hostname and port of the orderer from the `creds.json` file.
     - Replace `<CHANNEL_NAME>` with the name of the channel that the peer will join.
     - Replace `<ORGANIZATION_MSP_ID>` with the name of the organization from the `creds.json` file.
     - Replace `<PEER_ADDR>` with `localhost:7051`

   For example:

   ```
   export ORDERER_1=ash-zbc07b.4.secure.blockchain.ibm.com:21239
   export CHANNEL=defaultchannel
   export ORGID=PeerOrg1
   export PEERADDR=localhost:7051
   ```

4. Run the following peer CLI command to fetch the genesis block of the channel.

   **IMPORTANT:** In the following command, replace each occurence of `<PEER_ENROLL_ID>` with the enroll id associated with this  peer instance which was specified in the Quick Start template. This value can be located by running the command `ls /etc/hyperledger/`. There will be two folders listed: `fabric`, and the second one is your `<PEER_ENROLL_ID>`.

   ```
   CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/<PEER_ENROLL_ID>/tls/ca.crt CORE_PEER_TLS_ENABLED=true CORE_PEER_ADDRESS=${PEERADDR} CORE_PEER_LOCALMSPID=${ORGID} CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/<PEER_ENROLL_ID>/msp/ GOPATH=/ peer channel fetch 0 -o ${ORDERER_1} -c ${CHANNEL} --cafile /etc/hyperledger/<PEER_ENROLL_ID>/orderer_tlscacert.pem --tls
   ```
   {:codeblock}


   **Note:** It is possible when running any of these CLI commands that you may experience the following warning that can be safely ignored.

   ```
   [msp] getPemMaterialFromDir -> WARN 001 Failed reading file
   /etc/hyperledger/<PEER_ENROLL_ID>/msp/intermediatecerts/<intermediate cert name>.pem: no pem content for file  /etc/hyperledger/<PEER_ENROLL_ID>/msp/intermediatecerts/<intermediate cert name>.pem
   ```

   Verify that the command worked successfully and that genesis block is added to your peer container by running the following command:

   ```
   ls *.block
   ```
   {:codeblock}

   You know the genesis block is added successfully when you see something that is similar to the following example:

   ```
   defaultchannel_0.block
   ```

   Congratulations. Your {{site.data.keyword.blockchainfull_notm}} Platform for AWS peer is successfully connected to your {{site.data.keyword.blockchainfull_notm}} Platform network.

## FAQ
{: #remote-peer-aws-faq}

* **Q**. I encountered a CREATE_FAILED error when I launched the Quick Start.
* **A**. If AWS CloudFormation fails to create the stack, we recommend that you relaunch the template with Rollback on failure set to `No`. (This setting is under Advanced in the AWS CloudFormation console, Options page.) With this setting, the stack’s state will be retained and the instance will be left running, so you can troubleshoot the issue. (Look at the log files in `%ProgramFiles%\Amazon\EC2ConfigService` and `C:\cfn\log`.)

  - When you set Rollback on failure to `No`, you will continue to incur
  AWS charges for this stack. Please make sure to delete the stack when you finish troubleshooting. For additional information, see [Troubleshooting AWS CloudFormation ![External link icon](../images/external_link.svg "External link icon")](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/troubleshooting.html "Troubleshooting AWS CloudFormation") on the AWS website.

* **Q**. I encountered a size limitation error when I deployed the AWS Cloudformation templates.
* **A**. We recommend that you launch the Quick Start templates from the location we’ve provided or from another S3 bucket. If you deploy the templates from a local copy on your computer or from a non-S3 location, you might encounter template size limitations when you create the stack. For more information about AWS CloudFormation limits, see the [AWS documentation ![External link icon](../images/external_link.svg "External link icon")](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-limits.html "AWS CloudFormation Limits").

## What's next
{: #whats-next-aws}

After you set up the  peer in AWS, you can complete several operational steps before you can submit transactions to and read the distributed ledger from the blockchain network. For more information, see [Operating a  peer in AWS](remote_peer_operate_aws.html#remote-peer-operate-aws).

## High Availability (HA)
{: #aws-high-availability}

By default, for HA support, the Quick Start template deploys two instances of the peer, in two different availability zones.
To leverage this HA support, you also need to configure your [client applications for high availability](../v10_application.html#ha-app).

## Security Considerations
{: #remote-peer-aws-security}

The AWS Cloud provides a scalable, highly reliable platform that helps customers deploy applications and data quickly and securely. When you build systems on the AWS infrastructure, security responsibilities are shared between you and AWS. This shared model can reduce your operational burden as AWS operates, manages, and controls the components from the host operating system and virtualization layer down to the physical security of the facilities in which the services operate. In turn, you assume responsibility and management of the guest operating system (including updates and security patches), other associated applications, as well as the configuration of the AWS-provided security group firewall. For more information about security on AWS, visit the [AWS Cloud Security ![External link icon](../images/external_link.svg "External link icon")](https://aws.amazon.com/security/ "AWS Cloud Security").

### AWS Identity and Access Management (IAM)
{: #remote-peer-aws-iam}

This solution leverages an IAM role with least privileged access. It is not necessary or recommended to store SSH keys, secret keys, or accesskeys on the provisioned instances.

### OS security
{: #remote-peer-aws-ossecurity}

The root user on cluster nodes can be accessed only by using the SSH key specified during the deployment process. AWS doesn't store these SSH keys, so if you lose your SSH key you can lose access to these instances. Operating system patches are your responsibility and should be performed on a periodic basis.

### Security groups
{: #remote-peer-aws-securitygroups}

A security group acts as a firewall that controls the traffic for one or more instances. When you launch an instance, you associate one or more security groups with the instance. You add rules to each security group that allow traffic to or from its associated instances. You can modify the rules for a security group at any time. The new rules are automatically applied to all instances that are associated with the security group. The security groups created and assigned to the individual instances as part of this solution are restricted as much as possible while allowing access to the various functions needed by the peer. We recommend reviewing security groups to further restrict access as needed once the cluster is up and running.

### Peer Security
{: #aws-security}

The peers are deployed outside of {{site.data.keyword.blockchainfull_notm}} Platform; therefore, you are responsible for managing the security of the  peer. This includes important areas of security provided by Enterprise Plan networks such as key management and data encryption. Review the following topics when you consider security for your  peers.

#### Data security
{: #aws-security-data}

{{site.data.keyword.blockchainfull_notm}} Platform Enterprise Plan uses whole disk encryption that is based on [symmetric key encryption ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SSB23S_1.1.0.14/gtps7/s7symm.html "Symmetric cryptography") to protect all the data that the networks use. You must take similar steps in your own environment to protect your  peer data.

The data in your state database, whether you are using levelDB or couchDB, is not encrypted. You can use application level encryption to protect the data at rest in your state database.

#### Data residency
{: #aws-security-data-residency}

Data residency mandates that the processing and storage of all blockchain ledger data remains within the border of a single country.
For more details on how this can be accomplished, refer to this [topic](remote_peer.html#data-residency).

#### Key management
{: #aws-security-key-management}

Key management is a critical aspect of peer security. If a private key is compromised or lost, hostile actors might be able to access your  peer's data and functionality. {{site.data.keyword.blockchainfull_notm}} Platform Enterprise Plan uses [Hardware Security Modules](../glossary.html#hsm) (HSM) to store the private keys of your network. The HSM is a physical appliance that prevents other parties from accessing your private key.

When you deploy a  peer on AWS, you are responsible for managing your private keys. Although {{site.data.keyword.blockchainfull_notm}} Platform generates your private keys, those keys are not stored on the Platform. It is essential to ensure that you store your keys in a secure location so that they are not compromised. You can find the private key of your  peer in the keystore folder of your peer MSP, at the `/etc/hyperledger/<PEER_ENROLL_ID>/msp/keystore/` directory within your peer container. For more information about the certificates inside your  peer, visit the [Membership Services Provider](../certificates.html#msp) section of the [Managing certificates on {{site.data.keyword.blockchainfull_notm}} Platform](../certificates.html) topic.

You can use Key Escrow to recover lost private keys. This needs to be performed prior to the loss of any keys. If a private key cannot be recovered, you need to get new private keys by getting a new signCert from the Certificate Authority. You should also remove and replace your admin cert from any channels that you joined.

<!---
In IBP when a private key is created, two sets of independent key material is generated, in custody of two different entities. Those two sets of key materials are then combined to create the private key.
--->

#### TLS
{: #aws-security-tls}

[Transport Layer Security ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_7.1.0/com.ibm.mq.doc/sy10660_.htm "An overview of the SSL or TLS handshake") (TLS) is embedded in the trust model of Hyperledger Fabric. All components on {{site.data.keyword.blockchainfull_notm}} Platform use TLS to communicate authenticate and communicate with each other. Therefore, the network components on {{site.data.keyword.blockchainfull_notm}} Platform need to be able to complete a TLS handshake with your  peers. One implication of this is that you need to enable passthru, by using white listing for example, in your firewall from client apps to your peer.


#### Membership Service Provider configuration
{: #aws-security-MSP}

Components of IBM Blockchain Platform consume identities via Membership Service Providers (MSPs). MSPs associate the certificates that the CAs issue with network and channel roles. Refer to this [topic](../certificates.html#msp) for more information about how MSPs work with the  peer.

#### Application security
{: #aws-security-appl}

Because all chaincode invocations are signed, Fabric manages the application security. In addition, Fabric also includes ACL-based application level checks.

## License and pricing
{: #license-pricing-aws}

You must accept a Community edition license version of {{site.data.keyword.blockchainfull_notm}} Platform for AWS  to be able to use the deployment solution enabled by the Quick Start. The use of {{site.data.keyword.blockchainfull_notm}} Platform for AWS  (including all packages provided via the Quick Start offering, and packages derived from these) is not intended for production use. IBM may decide to de-authorize access to the code, and the use of this code, at any time.
The {{site.data.keyword.blockchainfull_notm}} Platform for AWS software license agreement contains more details about licensing terms. When you launch the Quick Start, you are asked to read and agree to the terms of the agreement.
