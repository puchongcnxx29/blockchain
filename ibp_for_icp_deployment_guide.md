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

# {{site.data.keyword.blockchainfull_notm}} Platform for ICP deployment guide

***[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

Blockchain networks that are based on [Hyperledger Fabric](https://hyperledger-fabric.readthedocs.io/en/release-1.2/) can be deployed in an almost limitless array of configurations to support as many use cases. Despite this malleability, a number of best practices exist, especially around the proper **sequence** for setting up and deploying network components.

This deployment guide shows the proper sequence for setting up a functioning {{site.data.keyword.blockchainfull}} Platform network on {{site.data.keyword.cloud_notm}} Private (ICP), in addition to best practices and things to think about when deploying. However, the basic rules still apply if you plan to set up only a functioning CA, orderer, or peer. Note that the SOLO ordering service, which deploys only a single orderer node, is not meant for production environments. Any network or channel that runs SOLO cannot be considered a "production" environment. However, it is possible to deploy the peer and CA in a production environment, especially when they are highly available.

**IMPORTANT**: The process for deploying IBP on ICP is difficult and presumes a high degree of expertise in Fabric. If you are new to Fabric, IBP, or ICP and your goal is to set up a development environment or proof of concept, consider checking out [Starter Plan](starter_plan.html) instead. Note also that not every potential deployment configuration is supported. Because of the difficulty, the assumption is that a Fabric expert will go through the process of deployment and of connecting components together before turning over administrative duties to other parties. Those experts are the target audience of this guide, and of the documentation we have for the Helm chart components in general.

## Deciding on your network configuration

The structure of a blockchain network must be dictated by the use case. These fundamental business decisions will vary in different situations, but let's consider a few.

* Setting up a **development environment**

  This is what [Starter Plan](starter_plan.html) is built for, but if you want to manage your own components in a development environment, you'll need a basic configuration that consists of an orderer and a peer (with one CA for each organization). For each org, you might decide to have only a single orderer and a single peer, similar to the configuration of the [fabcar network](https://hyperledger-fabric.readthedocs.io/en/release-1.2/understand_fabcar_network.html). Likewise, you might decide that you need only a single admin user for all of these components, rather than creating separate users for each component.

* Deploying **production components** either for a new network or to an existing network

  Production components, and production networks, have different needs than development environments or proofs of concept. For one thing, high availability becomes a priority. A SOLO ordering service, which includes only a single orderer (and is therefore a single point of failure) is by definition not meant for production.

**Note:** This deployment guide won't go through every iteration and potential network configuration, but does give common guidelines and rules to consider.

## Setting up an ICP Kubernetes cluster

After you decide the network structure, set up a Kubernetes cluster on ICP to accommodate your use cases. For more information, see [Setting up {{site.data.keyword.cloud_notm}} Private](ICP_setup.html).

## Setting up your CAs

In Fabric-based blockchain networks, the first component that must be deployed is a CA. This is because the configuration of a component must include at least one user identity that is authorized to operate it before the component is deployed.

Sometimes this can be difficult to understand conceptually. An electronic device such as a laptop, for example, can often be accessed by anyone when new. The first person who turns on the device is assumed to be the "owner" and is therefore allowed to register and establish passwords that every subsequent user must know. However, this is not how the components in Fabric-based blockchain networks are set up. The peers and orderers must have administrative information built into them when they're first deployed.

**It is recommended to deploy one CA per organization.** For example, if you deploy three peers that are associated with one organization and an orderer that is associated with another organization, you need to deploy two CAs. One CA is for the peer organization and the other one is for the orderer organization.

To prevent an infinite regression of CAs (where every CA must be linked to another CA, on and on forever) CAs have a default user name and password pair associated with them. You specify this user name and password when the CA is deployed. This CA is the "root CA", the root of trust for your component. In production networks, it's recommended to deploy at least one other CA that gets its certificate from this root CA and to use that CA, which is known as an "intermediate CA", to issue certificates to users and components. However, in this release, the use of intermediate CAs is not supported.

Every organization must have a CA for enrollment and a TLS CA.  When you deploy a CA on ICP, a TLS CA is also deployed in the same container by default. This TLS CA generates and manages your TLS certificates. The CA on a Starter or Enterprise plan network does not contain a TLS CA, but it does contain a CA that is used for enrollment. Therefore, if you plan to connect your peer to a Starter or Enterprise plan network,  before you deploy your peer, you must also [deploy a CA in ICP](howto/CA_deploy_icp.html) which will serve as the TLS CA for that peer. See also the [prerequisites for connecting an ICP peer to IBP](howto/peer_deploy_ibp#prerequisites-peer-ibp). Note that the TLS CA is only used for issuing certs and can be shutdown when that activity is completed.

For more information about TLS, see [Securing Communication With Transport Layer Security (TLS) ![External link icon](images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.3/enable_tls.html "Securing Communication With Transport Layer Security (TLS)") in Hyperledger Fabric Documentation.

### Preparing the MSPs for the orderer and peers

Because the IBP for ICP process is so elaborate, it's recommended to use a single admin user identity as the admin for all network component nodes during initial setup. This should cut down on deployment and connection errors by ensuring that one user can set up the configuration and connections between various components to ensure they're working properly. However, it's extremely important that each component have different certificates. Sometimes the distinction here can be easy to miss. The entity that signs a transaction proposal is not the admin of the peer but the **peer itself**. Therefore, the peer must be enrolled and have a certificate that it attaches to whatever it does, as well as a private key that it can use to generate certain kinds of signatures. For more information about identities and permissions in a Fabric-based blockchain network, see [Identity ![External link icon](images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.3/identity/identity.html "Identity") and [Membership ![External link icon](images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.3/membership/membership.html "Membership") in Fabric Documentation.

The Fabric CA client, which should be installed by following the instructions in [the documention on operating a CA](howto/CA_operate.html#fabric-ca-client), is used to register and enroll identities. This identity is the admin of the network components that you will deploy and allows you to change configurations and to connect components together. If you want to transfer the administration of these components to other users later on, you can register and enroll new admins for these components and remove yourself as an admin if necessary.

When the orderer or peer is deployed, an `init` container that is associated with the orderer or peer will use a Kubernetes secret object to create the MSP for the component. To learn how to create a secret object, see [Operating a CA](howto/CA_operate.html). As we said above, remember that you need to set up a CA and repeat this flow for each organization.

## Deploying orderers and peers

After a Kubernetes secret is created, you are ready to deploy a component. If your plan is to establish channels, it is recommended to deploy the orderer before any peers. Make sure to use different organization names for all of your components.

- **[Deploy the orderer](howto/orderer_deploy_icp.html)**. Note that at the present time, only the SOLO ordering service is supported. You do have options in the deployment process relating to compute.

- **[Deploy the peer to be connected to an ICP orderer](howto/peer_deploy_icp.html)**. A number of options for deploying your peer, including database type, can be specified in the Helm chart. Ensure that the peer's organization MSP ID is different than the orderer's organization MSP ID.

- **[Deploy the peer to be connected to an IBP network](howto/peer_deploy_ibp.html)**. The process for deploying a peer and connecting it to a [Starter Plan](starter_plan.html) or [Enterprise Plan](enterprise_plan.html) network in {{site.data.keyword.cloud_notm}} is different because it uses Connection Profiles and the Network Monitor UI. Note that the organization that the peer belongs to must already be joined to a channel in the network. Again, be sure the peer's organization MSP ID is different than the orderer's organization MSP ID.

## Growing your network

If your goal is to set up a development environment or proof of concept, you'll need to add peer organizations to the orderer system channel that is created during the deployment of the orderer. This is a multi-step process that utilizes every component and is documented in the relevant operational topics.

1. Before you can add an organization, you need to create it and set its MSP ID. See [Operating a CA](howto/CA_operate.html#deploy-orderer-peer).

2. After the organization has been created, it can be added to the orderer system channel. For more information, see [Operating an Orderer](howto/orderer_operate.html#add-organizations-to-consortium).

3. When an organization is listed on the orderer system channel, it is a member of the "consortium" and enabled to create an application channel, the kinds of channels that transactions happen on. For more information about how to create a channel, see [Operating a Peer](howto/peer_operate_icp.html#peer-icp-channeltx).
