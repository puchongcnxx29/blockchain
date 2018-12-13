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

# What's new
{: #whatsnew}

***[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

## December 7, 2018

The *Operate Starter Plan Network* and *Operate Enterprise Plan Network* topics are combined and replaced by a single tutorial about [Using the Network Monitor](v10_dashboard.html).

## November 27, 2018

{{site.data.keyword.blockchainfull}} Platform releases {{site.data.keyword.blockchainfull_notm}} Platform for {{site.data.keyword.cloud_notm}} Private (ICP). ICP is an application platform for developing and managing containerized applications that are based on Kubernetes and it allows users to deploy Certificate Authorities (CAs), orderers, and peers on x86, LinuxONE and IBM Z. {{site.data.keyword.blockchainfull_notm}} Platform for ICP is based on Hyperledger Fabric v1.2.1 and is deployed by using Kubernetes Helm charts. After you install the Helm chart, you can find it as a bundled service in the ICP Catalog. Note that [this offering](ibp-for-icp-about.html) is more suitable for experienced Fabric users.

For more information about ICP, see [{{site.data.keyword.cloud_notm}} Private v3.1.0 documentation ![External link icon](images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/kc_welcome_containers.html "{{site.data.keyword.cloud_notm}} Private v3.1.0 documentation").

The release of {{site.data.keyword.blockchainfull_notm}} Platform for ICP also marks the end of the IBM Blockchain Platform Remote Peer (Beta) program. It is still possible to deploy a peer in ICP or in AWS and connect it to an Starter Plan or Enterprise Plan network. For more information, see [deploying a peer against Starter or Enterprise Plan](howto/peer_deploy_ibp.html) and [deploying a peer in AWS](howto/remote_peer_aws.html). These peers are considered to be **distributed** peers instead of remote peers because the deployment is customer managed and as a result there is no central location to be remote from.

This release also debuts some improvements to the documentation table of contents. If you are a Starter or Enterprise user, your content can still be found in the **LEARN**, **HOW TO**, **REFERENCE**, and **HELP** sections, and the links are still the same. It might just be in a different subsection in the table of contents.

## November 13, 2018

{{site.data.keyword.blockchainfull_notm}} Platform releases {{site.data.keyword.blockchainfull_notm}} Platform for Amazon Web Services (AWS).

{{site.data.keyword.blockchainfull_notm}} Platform for AWS enables you to run peers in the AWS Cloud and connect the peers to existing blockchain networks in {{site.data.keyword.cloud_notm}}. Your peers in AWS can leverage the connection profile and other blockchain components in the existing networks, while giving you more flexibility to colocate your peers with other applications outside {{site.data.keyword.cloud_notm}}. For more information, see [About {{site.data.keyword.blockchainfull_notm}} Platform for Amazon Web Services](howto/remote_peer.html).

## October 4, 2018

{{site.data.keyword.blockchainfull_notm}} Platform upgrades Starter Plan from Hyperledger Fabric v1.1.0 to v1.2.1. For more information, see [About Starter Plan](starter_plan.html).

**Important:** Fabric v1.2.1 is currently not compatible with the Remote Peer beta, which uses a v1.1.0 peer. Starter Plan networks, which are created before October 4, 2018 and are based on Fabric v1.1.0, are not impacted and can still be used with the Remote Peer beta. {{site.data.keyword.blockchainfull_notm}} Platform will update the Remote Peer beta to use the v1.2.1 peer, which will be compatible with new Starter networks, which are at Fabric v1.2.1 level, and Enterprise networks, which are at Fabric v1.1.0 level. Until the Remote Peer beta is updated to Fabric v1.2.1, do not attempt to deploy a v1.1.0 remote peer with a new v1.2.1 Starter network.

## September 4, 2018

{{site.data.keyword.blockchainfull_notm}} Platform releases Remote Peer offering Beta. Remote Peer offering is based on Hyperledger Fabric v1.1.0. By using the Remote Peer, you can run {{site.data.keyword.blockchainfull_notm}} Platform peer nodes in your own IBM Cloud Private (ICP) or Amazon Web Services (AWS) Cloud environment. For more information, see [About remote peers](howto/remote_peer.html).

## June 15, 2018

{{site.data.keyword.blockchainfull_notm}} Platform releases Starter Plan GA. For more information, see [About Starter Plan](starter_plan.html).

## May 15, 2018

{{site.data.keyword.blockchainfull_notm}} Platform upgrades Enterprise Plan from Hyperledger Fabric v1.0.0 to v1.1.0. For more information, see [About Enterprise Plan](enterprise_plan.html).

## March, 18, 2018

{{site.data.keyword.blockchainfull_notm}} Platform releases Starter Plan Beta. Starter Plan offers you a development and testing environment to learn and practice in an {{site.data.keyword.blockchainfull_notm}} Platform network. For more information, see [About Starter Plan](starter_plan.html).

## August 11, 2017

{{site.data.keyword.blockchainfull_notm}} Platform replaces
{{site.data.keyword.blockchainfull_notm}} on Cloud and releases Enterprise Plan. For more information, see [About Enterprise Plan](enterprise_plan.html).
