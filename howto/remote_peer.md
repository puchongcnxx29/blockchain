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

# About {{site.data.keyword.blockchainfull_notm}} Platform for Amazon Web Services
{: #remote-peer-overview}

***[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

**Note:** The {{site.data.keyword.blockchainfull}} Platform Remote Peer on ICP (Beta) Program is ended. If you still want to run peers in your ICP environment, use the **{{site.data.keyword.blockchainfull_notm}} Platform for ICP** offering instead. For more information, see [About peers on ICP](../ibp-for-icp-about.html#ibp-icp-peer).

You can run the {{site.data.keyword.blockchainfull_notm}} Platform peer in the AWS Cloud after you connect it to an existing blockchain network in {{site.data.keyword.cloud_notm}}. Running peers outside {{site.data.keyword.cloud_notm}} provides more flexibility to grow or join a blockchain network while taking advantage of an existing network inside {{site.data.keyword.cloud_notm}}. Peers in the AWS Cloud leverage the Certificate Authorities (CAs) and ordering service on the platform, but allow you to collocate your peer with other applications outside {{site.data.keyword.cloud_notm}}.
{:shortdesc}


## Considerations
{: #remote-peer-limitations}

The {{site.data.keyword.blockchainfull_notm}} Platform for AWS does not have access to the full functionality or support of peers that are hosted on the {{site.data.keyword.blockchainfull_notm}} Platform. Before you run {{site.data.keyword.blockchainfull_notm}} Platform for AWS, ensure that you understand the following restrictions and limitations:
- Peers that run in other cloud environments are not visible in the Network Monitor of the blockchain network on {{site.data.keyword.cloud_notm}}.
- Peers running on {{site.data.keyword.blockchainfull_notm}} Platform for AWS cannot be addressed by using the Swagger UI in the Network Monitor UI.
- You are responsible for the management of health monitoring, security, logging, and resource usage of your {{site.data.keyword.blockchainfull_notm}} Platform for AWS peer nodes.
- You can connect your {{site.data.keyword.blockchainfull_notm}} Platform for AWS peers only to blockchain networks that are at Fabric level v1.1 or v1.2.1. You can find your Fabric version by opening the [Network Preferences window](../v10_dashboard.html#network-preferences) in your Network Monitor.
- The database type of the {{site.data.keyword.blockchainfull_notm}} Platform for AWS peer must match the database type of your blockchain network, either LevelDB or CouchDB.
- The CouchDB Fauxton interface is not available on the AWS peer.
- [Gossip](../glossary.html#gossip) for AWS peers is not currently supported. This implies that Fabric features that depend on gossip, such as [private data ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/private-data-arch.html "private data") and [service discovery ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/discovery-overview.html "service discovery"), are also not supported.

## Prerequisites
{: #remote-peer-prereq}

To use an {{site.data.keyword.blockchainfull_notm}} Platform for AWS peer, you must have an organization that is a member of a Starter Plan or Enterprise Plan network on the {{site.data.keyword.blockchainfull_notm}} Platform. The {{site.data.keyword.blockchainfull_notm}} Platform for AWS peer leverages the API endpoints, Hyperledger Fabric CAs, and ordering service of the {{site.data.keyword.blockchainfull_notm}} Platform network to operate. If you are not a member of any blockchain network, you need to create or join a network. For more information, see [Creating a network](../get_start.html#creating-a-network) or [Joining a network](../get_start.html#joining-a-network).

## License and pricing
{: #remote-peer-license-pricing}

{{site.data.keyword.blockchainfull_notm}} Platform for AWS is currently offered as a Community Edition, free of charge; in the future, IBP for AWS might change to a Bring-Your-Own-License (BYOL) model, which will require the purchase of a license from IBM.

**Note:** To operate an AWS peer, you must have an organization that belongs to a Starter Plan or Enterprise Plan network on {{site.data.keyword.blockchainfull_notm}} Platform. This implies that you or another member in the network must pay the {{site.data.keyword.blockchainfull_notm}} [membership fee](pricing.html#key-elements-of-pricing) for your organization. For more information about paying fees, see [Paying mode](paying_mode.html).


## Deploying an AWS peer
{: #deploy-remote-peer}

Use the AWS [Quick Start template ![External link icon](../images/external_link.svg "External link icon")](https://aws.amazon.com/quickstart/architecture/ibm-blockchain-platform/ "Quick Start Template") to easily deploy {{site.data.keyword.blockchainfull_notm}} Platform for AWS. For more information, see the [{{site.data.keyword.blockchainfull_notm}} Platform for AWS Quick Start Deployment Guide ![External link icon](../images/external_link.svg "External link icon")](https://s3.amazonaws.com/aws-quickstart/quickstart-ibm-fabric/doc/ibm-blockchain-platform-for-aws.pdf "IBM Blockchain Platform for AWS Quick Start Reference Deployment").

For instructions on how to deploy {{site.data.keyword.blockchainfull_notm}} Platform for AWS, see [Deploying peers in Amazon Web Services](remote_peer_aws.html "Deploying peers in Amazon Web Services").

The following diagram describes the process to deploy an {{site.data.keyword.blockchainfull_notm}} Platform for AWS peer.

<img usemap="#home_map1" border="0" class="image" id="image_ztx_crb_f1b2" src="../images/remote_peer_AWS_flow.png" width="750" alt="Click a box to get more details on the process." style="width:750px;" />
<map name="home_map1" id="home_map1">
<area href="remote_peer_aws.html#remote-peer-aws-account" alt="Configure or access AWS" title="Configure or access" shape="rect" coords="157.05, 52.53, 283.62, 127.11" />
<area href="remote_peer_aws.html#remote-peer-aws-account" alt="Create key pair" title="Create key pair" shape="rect" coords="300.97, 52.53, 427.54, 127.11" />
<area href="remote_peer_aws.html#prerequisites-aws" alt="Create or join a network" title="Create or join a network" shape="rect" coords="157.05, 131.8, 283.62, 206.37" />
<area href="remote_peer_operate_aws.html#aws-peer-operate-with-sdk" alt="Join a channel" title="Join a channel" shape="rect" coords="300.97, 131.8, 427.54, 206.37" />
<area href="remote_peer_aws.html#aws-register-peer" alt="Register peer identity" title="Register peer identity" shape="rect" coords="443.95, 131.8, 570.53, 206.37" />
<area href="remote_peer_aws.html#aws-network-endpoints" alt="Retrieve peer configuration information" title="Retrieve peer configuration information" shape="rect" coords="585.53, 131.8, 712.1, 206.37" />
<area href="remote_peer_aws.html#remote-peer-aws-launchqs" alt="Click link" title="Click link" shape="rect" coords="157.05, 258.43, 283.62, 333.48" />
<area href="remote_peer_aws.html#remote-peer-aws-launchqs" alt="Configure peer instances" title="Configure peer instances" shape="rect" coords="300.97, 258.43, 427.54, 333.48" />
<area href="remote_peer_aws.html#remote-peer-aws-test" alt="Verify deployment" title="Verify deployment" shape="rect" coords="443.95, 258.43, 570.53, 333.48" />
<area href="remote_peer_operate_aws.html#aws-peer-operate-with-sdk" alt="Use Fabric SDK" title="Use Fabric SDK" shape="rect" coords="157.05, 338.64, 283.62, 413" />
<area href="remote_peer_operate_aws.html#aws-peer-cli-operate" alt="Use Fabric tools CLI" title="Use Fabric tools CLI" shape="rect" coords="443.95, 338.64, 570.53, 413" />
</map>

*Figure 1. {{site.data.keyword.blockchainfull_notm}} Platform for AWS deployment flow on AWS*


## Operating an AWS peer
{: #operate-remote-peer}

After you deploy the AWS peer, you need to complete several operational steps before your peer can submit transactions to the network. The operational steps include adding your organization to a channel, joining your peer to the channel, installing chaincode on your peer, instantiating chaincode on the channel, and connecting applications to your peer. For more information, see [Operating peers in Amazon Web Service](remote_peer_operate_aws.html#remote-peer-operate-aws).

## Data residency
{: #data-residency}

Because blockchain networks are oblivious to the type of data that is processed, extra steps must sometimes be taken to keep certain kinds of data secure. The most common requirement on data residency is associated with laws within certain countries, which mandate that all data that is processed and stored in an IT system must remain within a specific country’s borders. Similarly, some companies in highly regulated industries, such as government, healthcare, and financial services, require that data must be stored entirely behind their firewall. Therefore, to achieve data residency, all components of the blockchain network must be part of the same [channel](../glossary.html#channel) and reside within the border of a single country.

To address data residency requirements, it is important to understand the Hyperledger Fabric architecture that underlies {{site.data.keyword.blockchainfull_notm}} Platform. The architecture is centered around three key components: Certificate Authority (CA), orderer, and peer. A peer receives ordered state updates in the form of blocks from the ordering service and maintains the state and the ledger. Therefore, a peer and an orderer have a direct relationship. The ledger contains the latest values for all keys and the data that the transaction logs include.

Additionally, client applications use the [Fabric SDKs](../v10_application.html#using-the-fabric-sdks) to send transactions to the peers and ordering service. These transactions include [read-write set ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/readwrite.html "Read-Write set semantics") data, which contains the key-value pairs on the ledger.

If in-country data residency is a requirement for your business, the orderer, peers, and client applications must reside in the same country. When an {{site.data.keyword.blockchainfull_notm}} Platform network is created in {{site.data.keyword.cloud_notm}}, you have the choice of selecting a location for the network. <!--For a Starter Plan network, you can select from US South, United Kingdom, and Sydney. For an Enterprise Plan network, you can select from currently available locations, which include Dallas, Frankfurt, London, Sao Paulo, Tokyo, and Toronto. -->For more information about regions and locations, see [{{site.data.keyword.blockchainfull_notm}} Platform regions and locations](../reference/ibp_regions.html). To achieve data residency in one of these countries, your peer should reside in the same country as the {{site.data.keyword.blockchainfull_notm}} Platform network location.

### A use case for data residency

Consider an {{site.data.keyword.blockchainfull_notm}} Platform network that includes the Orderer and Certificate Authority along with a consortium of four organizations. The organizations have one or more peer nodes. All four organizations are part of a single channel and all components of the network reside in the region (for example Frankfurt) where the {{site.data.keyword.blockchainfull_notm}} Platform network was deployed. Finally, the client applications that interact with the peers also reside within Germany. Data residency is maintained.  

![Data residency when all components are in the same country](../images/remote_peer_data_res_1.png "Data residency when all components are in the same country")  
*Figure 3. Data Residency when all components are in the same country*

Now let’s consider the implications when a **peer**  joins one of the organizations.  A peer can reside in the same region as the rest of the network or anywhere outside of the IBP network region:

-	If the peer resides in the same country as the rest of the network, data residency is maintained. All ledger data stays within Germany as in **Figure 3** above.
-	If the peer happens to reside in a different country (such as USA for example), data residency is no longer maintained as data on the peer ledger is shared outside the country's border.

To solve this problem **channels** can be used to segregate data to a subset of peers on the network. When the {{site.data.keyword.blockchainfull_notm}} Platform network contains peers and orderers across country borders, channels provide isolation of ledger data from organizations with peers outside the country border.  

**Note:** Orderers are always located in the data center region that you selected to host the network. It is not possible to have multiple orderers across country borders. However, peers can be located either in the data center or at a remote location outside {{site.data.keyword.cloud_notm}}.

![Data residency when peers are outside the country of the IBM Blockchain Platform region ](../images/remote_peer_data_res_2.png "Data Residency peers reside outside the country of the IBM Blockchain Platform region")  
*Figure 4. Data Residency peers reside outside the country of the IBM Blockchain Platform region*

In **Figure 4**, Data residency is not required for `OrgC` and `OrgD`. In fact, `OrgD` now includes two peers, `OrgD-peer1` and `OrgD-peer2`, which reside in the *United States*. Therefore, in order for  `OrgA`, `OrgB` and their respective client applications and peers that reside in Germany to isolate the ledger data on channel `X`, a new channel `Y` is created for `OrgC` and `OrgD`.

For a deeper understanding of the flow of data on the {{site.data.keyword.blockchainfull_notm}} Platform network, refer to the [Fabric documentation on transaction flow![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/txflow.html "Transaction Flow").

In the future, new technology in Hyperledger Fabric will improve the ability to achieve further data residency by using [Private data collections ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/private-data/private-data.html "Private data collections") and Zero Knowledge Proof.

- A Private data collection ensures that the private data is shared peer-to-peer (via the gossip protocol) to only the authorized peers to see it, for example, peers that are within the country borders. The data is stored in a private database on the peer.  The ordering service is not involved here and does not see the private data. A hash of that data is written to the ledgers of every peer on the channel. The hash that is used for state validation, serves as evidence of the transaction, and can be used for audit purposes. Private data is available for networks on the {{site.data.keyword.blockchainfull_notm}} Platform that are running on Fabric version 1.2.1. However, the private data feature is not available for peers.

- A Zero-Knowledge Proof (ZKP) allows a “prover” to assure a “verifier” that they have knowledge of a secret without revealing the secret itself. It's a way to show that you know something that satisfies a statement without showing what you know.

You can get more information about these technologies in the white paper about [Private and confidential transactions with Hyperledger Fabric ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/developerworks/cloud/library/cl-blockchain-private-confidential-transactions-hyperledger-fabric-zero-knowledge-proof/index.html "Private and confidential transactions with Hyperledger Fabric").

## Getting support
{: #remote-peer-support}

IBM Blockchain Platform does not provide support for this offering. If you encounter issues that are related to your peer, you can make use of free blockchain developer resources and support forums and get help from {{site.data.keyword.IBM_notm}} and the Fabric community. For more information, see [blockchain resources and support forums](../ibmblockchain_support.html#resources). You can also view the support resources on the **Get Help** screen of the Network Monitor.

- For issues that are related to AWS, you can use both [community support forums ![External link icon](../images/external_link.svg "External link icon")](https://forums.aws.amazon.com/index.jspa "AWS community support forums") and [AWS premium support ![External link icon](../images/external_link.svg "External link icon")](https://aws.amazon.com/premiumsupport/ "AWS premium support").

{{site.data.keyword.blockchainfull_notm}} does not support cases that are opened in {{site.data.keyword.cloud_notm}} and are related to {{site.data.keyword.blockchainfull_notm}} Platform for AWS. The Community Edition is meant for exploration, development, and testing, and do not use it for production.
