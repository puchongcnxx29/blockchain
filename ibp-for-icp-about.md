---

copyright:
  years: 2018
lastupdated: "2018-12-07"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# About {{site.data.keyword.blockchainfull_notm}} Platform for {{site.data.keyword.cloud_notm}} Private
{: #overview}

***[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

{{site.data.keyword.blockchainfull}} Platform releases IBM Blockchain Platform for {{site.data.keyword.cloud_notm}} Private (ICP), an application platform for developing and managing containerized applications based on Kubernetes which allows users to deploy Certificate Authorities (CAs), orderers, and peers on x86, and LinuxONE and IBM Z. {{site.data.keyword.blockchainfull_notm}} Platform for ICP is based on Hyperledger Fabric v1.2.1 and is deployed by using Kubernetes Helm charts.
{:shortdesc}

{{site.data.keyword.blockchainfull_notm}} Platform for {{site.data.keyword.cloud_notm}} Private is a bundled product for ICP customers to deploy blockchain components in their local environment. After you import the Helm chart, you can find it as an {{site.data.keyword.blockchainfull_notm}} Platform tile in the ICP Catalog. For more information about ICP, see the documentation for [{{site.data.keyword.cloud_notm}} Private version 3.1.0 ![External link icon](images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/kc_welcome_containers.html "{{site.data.keyword.cloud_notm}} Private 3.1.0").

## What {{site.data.keyword.blockchainfull_notm}} Platform for ICP offers

{{site.data.keyword.blockchainfull_notm}} Platform for IBM Cloud Private (ICP) allows you to deploy all of the fundamental components of a Hyperledger Fabric blockchain: A Certificate Authority, an ordering service, and peers. It provides you the flexibility to deploy different components depending on your business needs. You can use {{site.data.keyword.blockchainfull_notm}} for ICP to start a blockchain network by deploying and configuring an ordering service that binds organizations together in a blockchain consortium. You can also deploy peers and join other networks that use components based on Fabric, including {{site.data.keyword.blockchainfull_notm}} Platform networks deployed across clouds by using ICP, or Starter Plan and Enterprise Plan networks hosted on IBM Cloud. For more information about the building blocks of Hyperledger Fabric networks, see the [blockchain component overview](blockchain_component_overview.html).

## Is {{site.data.keyword.blockchainfull_notm}} Platform for ICP suitable for you

Running the {{site.data.keyword.blockchainfull_notm}} Platform components outside of {{site.data.keyword.cloud_notm}} provides you with more flexibility to grow or join a blockchain network. It helps network founders grow their networks by allowing new members to join while using the platform of their choice. It will allow organizations that are interested in joining blockchain networks collocate their peers with their existing applications or and integrate with your systems of record.

**Important:** The process for deploying {{site.data.keyword.blockchainfull_notm}} Platform for ICP is complicated and presumes a high degree of expertise in Fabric. If you are new to Fabric, ICP, or {{site.data.keyword.blockchainfull_notm}} Platform, and your goal is to set up a development environment or proof of concept, consider to use [Starter Plan](starter_plan.html) instead. Note also that not every potential deployment configuration is supported in {{site.data.keyword.blockchainfull_notm}} Platform for ICP.

Users of this offering will manage their own security and infrastructure. {{site.data.keyword.cloud_notm}} doesn't provide those services. Review the [Considerations and limitations](#ibp-icp-considerations) in the next section before you begin.

## Considerations and limitations
{: #ibp-icp-considerations}

Before getting started, ensure that you understand the following **considerations** and **limitations**:

- You are responsible for the health monitoring, security, logging, and managing the resource usage of your components.
- Components that run in other cloud environments are not visible in the Network Monitor of the networks running on {{site.data.keyword.cloud_notm}}.
- The Helm charts deploy a single instance of an orderer or peer, or CA.
- You can deploy multiple components to a single name space in ICP as long as they have different release names.
- Components cannot be addressed by using the Swagger UI in the Network Monitor UI.
- Mutual TLS is not supported.

**CA Considerations**
- This Helm chart deploys a single instance of the CA. As it is considered best practice to have a separate CA for each organization, it might be necessary to deploy several CAs. For example, if you plan to deploy one orderer and three peers, you will need at least two CAs (one for the orderer organization and another for the peer organization).
- Though you might choose to run a separate MySQL database, this option is not present in the Helm chart. The Helm chart will, however, deploy an SQLite database inside the CA to handle the database necessities of the CA, which include tracking the number of enrollments per user and any revoked certificates.

**Orderer Considerations**
- The ordering service is compatible with any component at v1.2 of Hyperledger Fabric.
- This Helm chart deploys a single instance of the SOLO ordering service (one orderer). Note that it is not possible to deploy more than one SOLO orderer on a channel to make the ordering service highly available. This is one reason why SOLO ordering services are considered to be for development, rather than production, environments. However, you can deploy multiple instances of the SOLO ordering service to different networks (that is, with a separate consortium).

**Peer Considerations**

- You can connect your peers only to blockchain networks that are at Fabric level v1.1 or v1.2.1. You can find your Hyperledger Fabric version by opening the [Network Preferences window](../v10_dashboard.html#network-preferences) in your Network Monitor. Follow the [instructions](#starter-enterprise-network-endpoints) for retrieving the peer connection information from Starter or Enterprise networks.
- The database type of the peer must match the database type of your blockchain network, either LevelDB or CouchDB.
- The CouchDB Fauxton interface is not available on the peer.
- [Gossip](../glossary.html#gossip) for peers is not currently supported. This implies that Fabric features that depend on gossip, such as [private data ![External link icon](images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/private-data-arch.html "private data") and [service discovery ![External link icon](images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/discovery-overview.html "service discovery"), are also not supported.

## System prerequisites
{: #prerequisites}

{{site.data.keyword.blockchainfull_notm}} Platform for ICP supports the following operating systems:
- Red Hat Enterprise Linux (RHEL) 7.3, 7.4, 7.5
- Ubuntu 18.04 LTS and 16.04 LTS
- SUSE Linux Enterprise Server (SLES) 12 SP3

The {{site.data.keyword.blockchainfull_notm}} Platform for ICP Helm chart has been validated to run on {{site.data.keyword.cloud_notm}} Private v3.1.0 clusters on Ubuntu Linux by using the following worker nodes and backing storage:

- **LinuxONE and IBM Z**: z/VM and KVM, which are using NFS.
- **x86**: Linux 64-bit that uses GlusterFS.

## License and pricing
{: #ibp-icp-license-pricing}
{{site.data.keyword.blockchainfull_notm}} Platform for {{site.data.keyword.cloud_notm}} Private consists of two editions, a paid offering that is downloadable from Passport Advantage and a free Community edition that is available in [GitHub ![External link icon](images/external_link.svg "External link icon")](https://github.com/IBM/charts/tree/master/stable/ibm-blockchain-platform-dev "IBM/charts").

### License
{: #ibp-icp-license}

{{site.data.keyword.blockchainfull_notm}} Platform on {{site.data.keyword.cloud_notm}} Private delivers the components needed to run a blockchain network on your own infrastructure and is deployed as an ICP application. You can access PPA and [download the Helm chart ](howto/helm_install_icp.html). Technical support from {{site.data.keyword.blockchainfull_notm}} is included in the purchase.

{{site.data.keyword.blockchainfull_notm}} Platform on IBM Cloud Private Community Edition is a free offering that is suitable for evaluation and experimentation and is deployed as an ICP application. Do not use the Community Edition for production. {{site.data.keyword.blockchainfull_notm}} Platform does not provide support for the Community Edition. You can access the [GitHub package ![External link icon](images/external_link.svg "External link icon")](https://github.com/IBM/charts/blob/master/repo/stable/ibm-blockchain-platform-dev-1.0.0.tgz "IBM/charts") and download the Helm chart.

### Pricing
{: #ibp-icp-pricing}

The pricing of {{site.data.keyword.blockchainfull_notm}} Platform on ICP is based on the number of Virtual Processor Cores (VPCs) used. A VPC can be either a virtual core that is assigned to a virtual server, or a physical processor core in a non-partitioned server. You must obtain a licensed entitlement for each VPC made available to the {{site.data.keyword.blockchainfull_notm}} Platform. <!-- A VPC is a unit of measurement by which a program can be licensed.-->

For more information about how to determine your usage of VPCs, see this article on ["Virtual Processor Cores (VPC)" ![External link icon](images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SS8JFY_9.2.0/com.ibm.lmt.doc/Inventory/overview/c_virtual_processor_core_licenses.html "Virtual Processor Cores (VPC's)") in IBM Knowledge Center. You can use the [IBM License Metric Tool](https://www.ibm.com/support/knowledgecenter/en/SS8JFY_9.2.0/com.ibm.lmt.doc/welcome/LMT_welcome.html) to configure and create a report that you can use to determine the number of VPCs that you need to license.


## Installing {{site.data.keyword.blockchainfull_notm}} Platform for ICP
{: #ibp-icp-install}

{{site.data.keyword.blockchainfull_notm}} Platform on {{site.data.keyword.cloud_notm}} Private is delivered as a Helm chart file that can be installed in a local ICP Cluster. After you install the Helm chart, you can find {{site.data.keyword.blockchainfull_notm}} Platform as an application in the ICP catalog.

- {{site.data.keyword.blockchainfull_notm}} Platform for ICP is delivered through Passport Advantage. You need to have the required license to access [Passport Advantage Online](https://www.ibm.com/software/passportadvantage/pao_customer.html). Upon your purchase, technical support for the {{site.data.keyword.blockchainfull_notm}} Platform is included.

- {{site.data.keyword.blockchainfull_notm}} Platform Community edition is meant for exploration, development, and testing. This free version of IBM Blockchain Platform for ICP can be accessed through GitHub. **Note:** {{site.data.keyword.blockchainfull_notm}} Platform does not provide support for the Community Edition.

For instructions on how to install the Helm chart and the necessary prerequisites, see [Installing {{site.data.keyword.blockchainfull_notm}} Platform on {{site.data.keyword.cloud_notm}} Private](howto/helm_install_icp.html).

If you are a new user of {{site.data.keyword.cloud_notm}} Private and would like information and tips on installing and deploying ICP, see [Setting up {{site.data.keyword.cloud_notm}} Private](ICP_setup.html).

After you install {{site.data.keyword.blockchainfull_notm}} Platform for ICP, you need to deploy each component of your network individually. You cannot deploy multiple components at the same time. Visit the [{{site.data.keyword.blockchainfull_notm}} Platform for ICP deployment guide](ibp_for_icp_deployment_guide.html) to learn the best practices for founding or joining a blockchain network before getting started. Then, review the steps to deploy and operate the individual components the sections below.

### Installing {{site.data.keyword.blockchainfull_notm}} Platform for ICP behind a firewall
{: #ibp-icp-firewall}

You can deploy the {{site.data.keyword.blockchainfull_notm}} Platform components behind a firewall, without having access to the public internet. The downloaded Helm chart package includes all of the Fabric component Docker images that {{site.data.keyword.blockchainfull_notm}} Platform uses, without pulling them from DockerHub during deployment.

The {{site.data.keyword.blockchainfull_notm}} Platform Community edition Helm chart package does not include the necessary Fabric component Docker images. It is configured to download those images from DockerHub during deployment and will fail without having access to the public internet. However, you can take a few additional steps to deploy the Community Edition components behind a firewall. For more information, see [Installing the Community Edition behind a firewall](howto/helm_install_icp.html#helm-install-prereqs-firewall)


## About Certificate Authorities on ICP
{: #ibp-icp-ca}

Certificate authorities (CAs) provide identity on the network. A CA can be considered as being similar to a notary that acts as an anchor of trust among multiple parties. All entities in the network are given a certificate signed by a root CA that encapsulates their digital identity. This certificate is the root of trust for all the sign and verify operations that are performed on the network. For more information about certificate authorities and the role they play in a blockchain network, see [Blockchain component overview](blockchain_component_overview.html).

The CA will validate the identity and issue certificates for the other components in your network that you need to deploy. As a result, you need to deploy a CA for your organization before you deploy other components.

- To learn how to configure and deploy a CA after installing the Helm chart, see [Deploying a Certificate Authority in {{site.data.keyword.cloud_notm}} Private](howto/CA_deploy_icp.html).

- To learn how to use your CA to generate certificates and complete the prerequisite steps to deploy additional components, see [Operating a Certificate Authority in {{site.data.keyword.cloud_notm}} Private](howto/CA_operate.html).

## About orderers on ICP
{: #ibp-icp-orderer}

Orderers authenticate clients, order transactions, and broadcast transactions in a blockchain network. They are the common binding of blockchain networks based on Hyperledger Fabric. As a result, the organization that founds a network needs to deploy and start an "ordering service" (the node or collection of nodes that do the ordering) before other organizations can deploy their peers, join channels, and start transacting on the network. For more information about orderers and the role that they play in a blockchain network, see [Blockchain component overview](blockchain_component_overview.html#orderer).

If you are founding a blockchain network, you need to deploy an orderer. After it has been deployed, you can invite other organizations to your consortium, which can then create channels of their own.

- To learn how to configure and deploy an orderer after you install the Helm chart, see [Deploying an orderer in ICP](howto/orderer_deploy_icp.html).

- To learn how to build a consortium, see [Operating an orderer on ICP](howto/orderer_operate.html).

## About peers on ICP
{: #ibp-icp-peer}

Peers are a fundamental element of the network because they host ledgers and smart contracts. Smart contracts and ledgers are used to encapsulate the shared processes and shared information in a network, respectively. For more information about peers and the role that they play in a blockchain network, see [Blockchain component overview](blockchain_component_overview.html).

- When you are ready to join a network, you can deploy a peer that will join channels, endorse transactions, and store the channel ledgers. For information about deploying a peer on ICP that will connect to other components on ICP, see [Deploying a peer on ICP](howto/peer_deploy_icp.html). For information on deploying a peer on ICP that will connect to a Starter or Enterprise Plan network, see [Deploying a peer that will connect to Starter or Enterprise](howto/peer_deploy_ibp.html).

- After you set up the peer, you need to complete several operational steps before you can submit transactions and read the distributed ledger from the blockchain network.

  - If you are connecting your peer to a {{site.data.keyword.blockchainfull_notm}} Platform deployed on ICP, see [Operating peers on {{site.data.keyword.cloud_notm}} Private with a Multi-cloud Network](howto/peer_operate_icp.html).
  - If you are connecting your peer to a Starter Plan or Enterprise Plan network that is deployed on {{site.data.keyword.cloud_notm}}, see [Operating peers on {{site.data.keyword.cloud_notm}} Private with Starter Plan or Enterprise Plan](howto/peer_operate_ibp.html).

## Security Considerations
{: #icp-security}

Because these components are deployed outside of {{site.data.keyword.cloud_notm}}, you are responsible for managing their security. This includes important areas of security, such as key management and data encryption. Review the following topics when you consider security for your components.

### Data security
{: #icp-security-data}

{{site.data.keyword.blockchainfull_notm}} Platform Starter and Enterprise plans use whole disk encryption that is based on [symmetric key encryption ![External link icon](images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SSB23S_1.1.0.14/gtps7/s7symm.html "Symmetric cryptography") to protect all the data that the networks use. You must take similar steps in your own environment to protect your peer data.

The data in your state database, whether you use LevelDB or CouchDB, is not encrypted. You can use application level encryption to protect the data at rest in your state database.

### Data residency
{: #data-residency}

Data residency requirements can mandate that the processing and storage of all blockchain ledger data remain within the border of a single country (or within some other defined boundary). For more information on how data residency can be accomplished, see [Data residency](#icp-data-residency).

### Key management
{: #icp-security-key-management}

Key management is a critical aspect of security. If a private key is compromised or lost, hostile actors might be able to access your data and functionality. IBM uses physical appliances that are called [Hardware Security Modules](../glossary.html#hsm) (HSM) to store the private keys of {{site.data.keyword.blockchainfull_notm}} Platform Enterprise Plan networks.

When you deploy a component on ICP, you are responsible for managing your private keys. Although {{site.data.keyword.blockchainfull_notm}} Platform generates private keys, those keys are not stored on the Platform. It is essential to store your keys securely so that they are not compromised. You can find the private key of your component in the keystore folder of your peer MSP, at the `/mnt/crypto/peer/peer/msp/keystore/` directory within your component. For more information about the certificates inside your peer, see the [Membership Services Provider](../certificates.html#msp) section of the [Managing certificates on {{site.data.keyword.blockchainfull_notm}} Platform](../certificates.html) tutorial.

You can use Key Escrow to recover lost private keys. This needs to be performed prior to the loss of any keys. If a private key cannot be recovered, you need to get new private keys by registering a new identity with your Certificate Authority. You should also remove and replace your signCert from any channels that you joined.

### TLS
{: #icp-security-tls}

[Transport Layer Security ![External link icon](images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_7.1.0/com.ibm.mq.doc/sy10660_.htm "An overview of the SSL or TLS handshake") (TLS) is embedded in the trust model of Hyperledger Fabric. All Starter and Enterprise components on {{site.data.keyword.blockchainfull_notm}} Platform use TLS to authenticate and communicate with each other. Therefore, the network components on Platform need to be able to complete a TLS handshake with your peers. One implication of this is that you need to enable passthru, by using white listing for example, in your firewall from client apps to your peer.

<!--
You can use Mutual TLS, which requires two way (server-client) rather than one way (server only) authentication, to secure the communication between your application and Enterprise Plan networks. You can use the Network Monitor [to enable mutual TLS](../v10_dashboard.html#mutual-tls) for peers on {{site.data.keyword.blockchainfull_notm}} Platform. To enable Mutual TLS on your peer, follow the instructions to [enable mutual-TLS for peer nodes ![External link icon](images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.1/enable_tls.html "Securing Communication with Transport Layer Security") in the Hyperledger Fabric documentation. It is strongly recommended that you enable mutual-TLS for your applications.
-->

### Membership Service Provider configuration
{: #icp-security-MSP}

Components of {{site.data.keyword.blockchainfull_notm}} Platform consume identities via Membership Service Providers (MSPs). MSPs associate the certificates that the CAs issue with network and channel roles. For more information about how MSPs work with the peer, see [Membership Service Providers (MSPs)](../certificates.html#msp).

### Application security
{: #icp-security-appl}

Because all chaincode invocations are signed, Fabric manages the application security. In addition, Fabric also includes ACL-based application level checks.

## Data residency
{: #icp-data-residency}

Because blockchain networks are oblivious to the type of data that is processed, extra steps must sometimes be taken to keep certain kinds of data secure. The most common requirement on data residency is associated with laws within certain countries, which mandate that all data that is processed and stored in an IT system must remain within a specific country’s borders. Similarly, some companies in highly regulated industries, such as government, healthcare, and financial services, require that data must be stored entirely behind their firewall. Therefore, to achieve data residency, all components of the blockchain network must be part of the same [channel](glossary.html#channel) and reside within the border of a single country.

To address data residency requirements, it is important to understand the Hyperledger Fabric architecture that underlies {{site.data.keyword.blockchainfull_notm}} Platform. The architecture is centered around three key components: an ordering service (made up of orderers), Certificate Authorities (CA), and peers. A peer receives ordered state updates in the form of blocks from the ordering service and maintains the state and the ledger. Therefore, a peer and an ordering service have a direct relationship. The ledger contains the latest values for all keys and the data that the transaction logs include.

Additionally, client applications use the [Fabric SDKs](v10_application.html#using-the-fabric-sdks) to send transactions to the peers and ordering service. These transactions include [read-write set ![External link icon](images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/readwrite.html "Read-Write set semantics") data, which contains the key-value pairs on the ledger.

If in-country data residency is a requirement, the orderer, peers, and client applications must reside in the same country. When an {{site.data.keyword.blockchainfull_notm}} Platform network is created in {{site.data.keyword.cloud_notm}}, you have the choice of selecting a location for the network. <!--For a Starter Plan network, you can select from US South, United Kingdom, and Sydney. For an Enterprise Plan network, you can select from currently available locations, which include Dallas, Frankfurt, London, Sao Paulo, Tokyo, and Toronto. -->For more information about regions and locations, see [{{site.data.keyword.blockchainfull_notm}} Platform regions and locations](reference/ibp_regions.html).

### A use case for data residency

Consider an {{site.data.keyword.blockchainfull_notm}} Platform network that includes the orderer and CA along with a consortium of four organizations. The organizations have one or more peer nodes, and all four organizations are part of a single channel and all components of the network reside in the region (for example Frankfurt) where the {{site.data.keyword.blockchainfull_notm}} Platform network was deployed. Finally, the client applications that interact with the peers also reside within Germany.

In this example, data residency is maintained.

![Data residency when all components are in the same country](images/remote_peer_data_res_1.png "Data residency when all components are in the same country")
*Figure 1. Data Residency when all components are in the same country*

Now let’s consider the implications when a **distributed peer** joins one of the organizations. A distributed peer can reside in the same region as the rest of the network or anywhere outside of the region of the {{site.data.keyword.blockchainfull_notm}} Platform network:

-	If the peer resides in the same country as the rest of the network, data residency is maintained. All ledger data stays within Germany as in **Figure 1** above.
-	If the peer happens to reside in a different country (such as USA for example), data residency is no longer maintained as data on the peer ledger is shared outside the country's border.

To solve this problem **channels** can be used to segregate data to a subset of peers on the network. When the {{site.data.keyword.blockchainfull_notm}} Platform network contains a distributed peer and orderers across country borders, channels provide isolation of ledger data from organizations with peers outside the country border.

**Note:** Orderers are always located in the data center region you selected to host the network. It is not possible to have multiple orderers across country borders. Peers, however, can be located either in the data center or at a remote location outside {{site.data.keyword.cloud_notm}}.

![Data residency when peers are outside the country of the IBM Blockchain Platform region](images/remote_peer_data_res_2.png "Data Residency peers reside outside the country of the IBM Blockchain Platform region")
*Figure 2. Data Residency peers reside outside the country of the IBM Blockchain Platform region*

In **Figure 2**, Data residency is not required for `OrgC` and `OrgD`. In fact, `OrgD` now includes two distributed peers, `OrgD-peer1` and `OrgD-peer2`, which reside in the *United States*. Therefore, in order for  `OrgA`, `OrgB`, and their respective client applications and peers that reside in Germany to isolate the ledger data on channel `X`, a new channel `Y` is created for `OrgC` and `OrgD`.

For a deeper understanding of the flow of data on the {{site.data.keyword.blockchainfull_notm}} Platform network, see the [Fabric documentation on transaction flow![External link icon](images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/txflow.html "Transaction Flow").

In the future, new technology in Hyperledger Fabric will improve the ability to achieve further data residency by using [Private data collections ![External link icon](images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/private-data/private-data.html "Private data collections") and zero knowledge proof.

- A private data collection ensures that the private data is shared peer-to-peer (via the gossip protocol) to only the authorized peers to see it, for example, peers that are within the country borders. The data is stored in a private database on the peer. The ordering service is not involved here and does not see the private data. A hash of that data is written to the ledgers of every peer on the channel. The hash that is used for state validation, serves as evidence of the transaction, and can be used for audit purposes. Private data is available for networks on the {{site.data.keyword.blockchainfull_notm}} Platform that are running on Fabric version 1.2.1. However, the private data feature is not available for peers running in ICP, as ICP networks currently do not use gossip.

- A zero knowledge proof (ZKP) allows a “prover” to assure a “verifier” that they have knowledge of a secret without revealing the secret itself.

You can get more information about these technologies in the white paper about [Private and confidential transactions with Hyperledger Fabric ![External link icon](images/external_link.svg "External link icon")](https://www.ibm.com/developerworks/cloud/library/cl-blockchain-private-confidential-transactions-hyperledger-fabric-zero-knowledge-proof/index.html "Private and confidential transactions with Hyperledger Fabric").

## Getting support
{: #ibp-icp-support}

For information on digital support offerings, refer to the IBM Blockchain Platform support [resources and support forums](ibmblockchain_support.html#resources).

### {{site.data.keyword.blockchainfull_notm}} Platform for ICP

If you have purchased an IBM Blockchain Platform for ICP license and you want to contact Customer support, see the information about [accessing the IBM Support Community and opening a support ticket ![External link icon](images/external_link.svg "External link icon")](https://www-01.ibm.com/support/docview.wss?uid=ibm10740041 "{{site.data.keyword.blockchainfull_notm}} Platform for ICP Support").

### {{site.data.keyword.blockchainfull_notm}} Platform Community Edition

The Community Edition is intended for exploration, development, and testing. IBM Blockchain Platform does not provide support for the {{site.data.keyword.blockchainfull_notm}} Platform Community Edition.

If you encounter issues that are related to your blockchain components, you can use free blockchain developer resources and support forums and get help from IBM and the Fabric community. For more information, see [blockchain resources and support forums](ibmblockchain_support.html#resources).

For issues that are related to IBM Cloud Private, you can take advantage of both [free digital support and paid support that ICP offers](https://www.ibm.com/developerworks/community/blogs/fe25b4ef-ea6a-4d86-a629-6f87ccf4649e/entry/Learn_more_about_IBM_Cloud_Private_Support?lang=en_us).
