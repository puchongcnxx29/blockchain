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

# {{site.data.keyword.blockchainfull_notm}} Platform

***[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

**ATTENTION:** Before you use an {{site.data.keyword.blockchainfull}} Platform offering, read the technical and support information in the [Disclaimer](needtoknow.html) section.
{:shortdesc}

{{site.data.keyword.blockchainfull_notm}} Platform (IBP) is the only integrated business-ready platform that addresses the full lifecycle of a multi-organization blockchain network. It is designed to accelerate, through collaboration in each phase, the creation of "built for business" global blockchain networks with the performance and security for even the most demanding use cases and regulated industries. {{site.data.keyword.IBM_notm}}-managed network offerings on {{site.data.keyword.cloud_notm}} are suitable for customers who are new to blockchain. Customers with experiences in using Hyperledger Fabric can deploy network components or a network on their own infrastructure through {{site.data.keyword.cloud_notm}} Private (ICP) or Amazon Web Services (AWS).

{{site.data.keyword.blockchainfull_notm}} Platform provides different offerings to help all types of users get started on their blockchain journey and move their applications into production.

|       | [**Starter Plan**](#starter-enterprise)      | [ **Enterprise Plan**](#starter-enterprise)  | [**IBP for ICP**](#ibp-for-icp) | [**IBP for AWS**](#ibp-for-aws)|
| ------------------------- |--------------------------|-----|-----|------|
| **What's included** | **Basic service levels, development, and testing environment** | **Advanced service levels and enterprise production ready environment** |**Deployable CA, Orderer, and Peer Helm charts** | **AWS Quick Start Peer**  |
| **Billing policy** | **Monthly subscription with [available cloud credits](howto/pricing.html#starter-plan-pricing)** | **Monthly subscription** |  **[VPC pricing](ibp-for-icp-about.html#ibp-icp-pricing) and Free Community Edition** | **Free** |
| **Cloud platform**| **IBM Cloud**|**IBM Cloud**|**IBM Cloud Private**| **AWS**|

**Caution:** Do not use **Starter Plan** for production usage. It is a development and testing environment, and is not suitable for production workloads.

## {{site.data.keyword.blockchainfull_notm}} Platform capabilities

{{site.data.keyword.blockchainfull_notm}} offerings are built on the [Hyperledger Fabric](reference/v10_fabric.html) code base that leverages a modular architecture to achieve enterprise levels of security, data integrity, scalability, and performance to meet your business needs.

{{site.data.keyword.blockchainfull_notm}} Platform provides a highly secure and permissioned blockchain network, upon which authenticated members can easily define assets and create the business solutions for modifying and exchanging them. With {{site.data.keyword.blockchainfull_notm}} Platform offerings, you can take advantage of an orchestration framework that allows you to **quickly organize your consortium into an active blockchain network**. {{site.data.keyword.blockchainfull_notm}} Platform provides friendly tooling that is designed to make it simple for multiple institutions to join together and create a democratically governed network. The creation, governance, and operation tasks of the network become intuitive and transparent through a built-in dashboard monitor and provisioned utilities. By foregoing the otherwise cumbersome process of creating a network and implementing governance, consortium members can instead **focus on the deployment of smart contracts and transfer of assets and information**.

**High availability** for the integral resources of the network (peers, orderers, Certificate Authorities) eliminates the crippling effects that can arise from single points of failure. A built-in dashboard monitor allows for easy management of these resources and provides a powerful mechanism to visualize assets and smart contracts. *Only supported on Starter and Enterprise offerings*.

The **modularity** of the Hyperledger Fabric architecture and the distinct separation of network roles provide an infrastructure that enables scalability and adaptability to a wide variety of use cases. *Available and supported on all {{site.data.keyword.blockchainfull_notm}} Platform offerings*.

The checks and balances that occur throughout the lifecycle of a transaction ensure consistent and fully vetted results, and ledgers constantly stay synchronized through an implementation of gossip protocol. Identity and access control are easily enforced through **sign/verify** operations that occur perpetually throughout the network, which are *available and supported on all {{site.data.keyword.blockchainfull_notm}} Platform offerings*.

**Governance tooling** is provided to allow members to administer and manage the critical business rules for their network. For example, you might want to implement a policy that defines how many members of a network must agree in order for a new member to join. Or perhaps there is an asset that requires endorsement from every participant in order for a modification to take place. Rules of governance are fundamental for any type of business network and are often extremely elaborate. Governance tooling (policy editors for example) greatly simplifies this process. *Only available on Starter and Enterprise offerings*.

Enterprise Plan networks run in a **highly secure and isolated** environment with no external access (including root access) to network resources. Data is encrypted in flight and at rest, and hardware security support modules allow digital keys to be protected in accordance with industry regulations. Hardware virtualization is used to run each node in an isolated environment, and therefore protecting other nodes in the network from peers with potentially misbehaving or malicious chaincode. Hashing, sign/verify operations, and node-to-node communications are accelerated thanks to advanced cryptography implementations. *Only available with Enterprise*.

For more details about all Hyperledger Fabric features and functionality,
see [Hyperledger Fabric documentation ![External link icon](images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/latest/ "Hyperledger Fabric documentation").


<!--

## Develop applications to run on the network

Explore and accelerate blockchain development by leveraging best practices honed from more than 400 client engagements:

* Ensure close alignment across businesses with technology that significantly reduces blockchain application development time (applications that would otherwise take six weeks to develop can be created in less than two days).
* Quickly build blockchain skills within your existing roster of programmers by leveraging {{site.data.keyword.blockchainfull_notm}} developer tools.
* Give developers the flexibility to learn and develop in their preferred environment with an open and modern toolset.

As a business owner, you can develop your use case with help from IBM's deep bench of industry and blockchain experts who come together in the {{site.data.keyword.blockchainfull_notm}} Garage to harness the full power of the {{site.data.keyword.blockchainfull_notm}} Platform.

As a developer, you can quickly and easily align business requirements and accelerate blockchain application development in  {{site.data.keyword.blockchainfull_notm}} Platform network environment by using an interactive playground to develop, iterate, and test business networks. These tools are designed to turn business rules into business network code in your preferred environment:

* **Explore online**
  Leverage [{{site.data.keyword.blockchainfull_notm}} Platform: Develop](./develop.html) which is powered by an open source development tool to learn key blockchain concepts, create network definitions, and leverage reusable industry models and smart contract libraries.

  After you developed your business network, you can deploy it to a live network that runs on the {{site.data.keyword.blockchainfull_notm}} Platform. For more information, see [Deploying business networks with Starter Plan](./develop_starter.html) and [Deploying business networks with Enterprise Plan](./develop_enterprise.html).

* **Install locally**
  Leverage Hyperledger Fabric to develop and test directly on your laptop. For more information, see [Building Your First Network](http://hyperledger-fabric.readthedocs.io/en/release-1.2/build_network.html).

* **Collaborate in a cloud environment**
  Use ready-to-use live networks with Starter Plan and Enterprise Plan options to develop and share your code with others. For more information, see [About Starter Plan](starter_plan.html) and [About Enterprise Plan](enterprise_plan.html).

-->

## Starter Plan and Enterprise Plan
{: #starter-enterprise}

Starter and Enterprise plans enable you to deploy and operate decentralized blockchain networks with a production-ready secure service. **Starter Plan** is an environment to learn or get started developing blockchain networks and **Enterprise Plan** is a production environment that offers high levels of security and support. Start small and scale your network elastically as your membership and transaction volumes increase by taking advantage of the following features:

* An ultra high security environment with many hardware, firmware, and software security features.
* Hardened architecture for scalability, resiliency, and availability.
* Optimized for performance and running on the world's fastest Linux compute.

Operating your network on the {{site.data.keyword.blockchainfull_notm}} Platform includes tools and capabilities to simplify administrative tasks:

* Dashboards for monitoring and managing the resources on the network.
* Seamless upgrades of the full code stack.
* 24/7 technical support that is integrated into the portal.
* Hardened security stack with no privileged access, malware and tamper resistance, 100% encryption, and many more features for networks with sensitive data in regulated industries.

Starter Plan and Enterprise Plan are platform services on {{site.data.keyword.cloud_notm}} and both plans follow the [{{site.data.keyword.cloud_notm}} Services terms ![External link icon](images/external_link.svg "External link icon")](https://www-03.ibm.com/software/sla/sladb.nsf/sla/bm-6605-13 "{{site.data.keyword.cloud_notm}} Services terms") on service level agreements (SLAs). Note that both Starter and Enterprise plan networks are provisioned across **multiple environments** in geographically separated data-centers.

For more information about Starter Plan and Enterprise Plan, see [About Starter Plan](starter_plan.html) and [About Enterprise Plan](enterprise_plan.html). If you are ready to start, sign up now for your [IBP Starter or Enterprise membership ![External link icon](images/external_link.svg "External link icon")](https://console.bluemix.net/catalog/services/blockchain?env_id=ibm:yp:us-south&taxonomyNavigation=apps) on {{site.data.keyword.cloud_notm}}!

## {{site.data.keyword.blockchainfull_notm}} Platform for ICP
{: #ibp-for-icp}

{{site.data.keyword.blockchainfull_notm}} Platform for ICP delivers the components that you need to run a blockchain network on your own infrastructure through ICP. The components include Hyperledger Fabric a Certificate Authority (CA), an orderer, and a peer, which you deploy, manage, and set up by using Kubernetes Helm charts. **This offering is intended for customers with advanced Hyperledger Fabric experience.**

IBP for ICP enables blockchain networks to be deployed on a private cloud to address data residency requirements, market regulations, and infrastructure preference. It simplifies the deployment of essential elements of a blockchain network in your own infrastructure through {{site.data.keyword.cloud_notm}} Private, a Kubernetes-based application platform for developing and managing on-premises, containerized applications.

 * Enables clients to manage IBP networks with their own infrastructure. A free Community Edition allows customers to run in their own isolated and secure environments, but no support will be provided.
 * Enables clients to configure Fabric on Kubernetes by using Helm charts and detailed documentation for operations.
 * Entitles clients with advanced technical support, unless you use the Community Edition.

 For more information about IBP for ICP, see [About {{site.data.keyword.blockchainfull_notm}} Platform for {{site.data.keyword.cloud_notm}} Private](ibp-for-icp-about.html).

## {{site.data.keyword.blockchainfull_notm}} Platform for AWS
{: #ibp-for-aws}

{{site.data.keyword.blockchainfull_notm}} Platform for AWS enables peers to leverage the connection profile, Hyperledger Fabric certificate authorities (CAs), and the ordering service of an existing Starter or Enterprise Plan network on {{site.data.keyword.cloud_notm}} to process transactions via an AWS Quick Start template. The Quick Start allows you to deploy peers by using AWS CloudFormation templates. This template is for IT infrastructure decision makers and system administrators who want to rapidly configure, deploy, and run AWS-hosted IBP peers that are connected to a Starter or Enterprise plan network. You can use the template to build a new virtual private cloud (VPC) in AWS, or deploy the peer into an existing VPC.

The Quick Start completes the following configurations:
 * A highly available architecture that spans two Availability Zones.
 * A VPC configured with public subnets according to AWS best practices. This provides you with your own virtual network on AWS.
 * An internet gateway to allow access to the internet.
 * In the public subnets, two peers across two Availability Zones (one peer in each subnet).
 * In each public subnet, a peer container with either an embedded LevelDB database or a secondary CouchDB container.

For more information about IBP for AWS, see [About {{site.data.keyword.blockchainfull_notm}} Platform for Amazon Web Services](howto/remote_peer.html).

## {{site.data.keyword.IBM_notm}} support

{{site.data.keyword.IBM_notm}} offers various support options on {{site.data.keyword.IBM_notm}} implemented blockchain solutions. For more information about {{site.data.keyword.blockchainfull_notm}} Support, see [Getting support](ibmblockchain_support.html).
