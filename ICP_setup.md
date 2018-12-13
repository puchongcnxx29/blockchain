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

# Setting up IBM Cloud Private (ICP)


***[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


Before you deploy {{site.data.keyword.blockchainfull}} Platform components and build blockchain network on {{site.data.keyword.cloud_notm}} Private (ICP), you need to set up ICP in your own environment.
{:shortdesc}

## Prerequisites
{: #icp-setup-prerequisites}
Complete the following prerequisites and prepare your environment to install ICP.

### Docker
{{site.data.keyword.cloud_notm}} Private requires Docker to be installed. Follow the related instructions in [Installing {{site.data.keyword.cloud_notm}} Private ![External link icon](/images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/installing/install.html "Installing {{site.data.keyword.cloud_notm}} Private") to install Docker.

### ICP settings
Before you install ICP, the following tips are useful to prepare your nodes for ICP installation. Additional ICP prerequisites can be found in the [ICP documentation ![External link icon](/images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/installing/prep.html "Preparing your cluster for installation").

#### Update `vm.max_map_count` setting
ICP uses Elastic Search for logging and metering. To avoid out-of-memory exceptions, Elastic Search requires the `vm.max_map_count` system property to be configured. Before you install ICP, see the [Elastic Search configuration instructions ![External link icon](/images/external_link.svg "External link icon")](https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html "Virtual memory") to configure this property on each node. You can use the following commands to set this property permanently:

```
sysctl -w vm.max_map_count=262144; sysctl vm.max_map_count
echo "vm.max_map_count=262144” | tee -a /etc/sysctl.conf
```
{:codeblock}

#### Configure the `/etc/hosts` file on each node in your cluster

- ICP uses [Kubernetes ![External link icon](images/external_link.svg "External link icon")](https://kubernetes.io/docs/tutorials/kubernetes-basics/ "Learn Kubernetes Basics") to manage containerized applications. The Kubernetes Domain Name Server (DNS) fails if host names are not configured in the `/etc/hosts` file on each node. [Insert the IP address, hostname, and shortname of each node in your cluster ![External link icon](images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/installing/prep_cluster.html "Configuring your cluster") into the `/etc/hosts` file on each node.

- [IPv6 is not supported by ICP ![External link icon](images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/getting_started/known_issues.html#ipv6 "IPv6 is not supported"). To avoid problems with the DNS service in an ICP cluster, disable the IPv6 settings in the `/etc/hosts` file on each node by commenting out the following line with a `#` sign at the beginning of the line:
  ```
  #::1  localhost ip6-localhost ip6-loopback
  ```
  {:codeblock}

### Resources required
{: #helm-icp-resources}

Ensure that your ICP system meets the minimum hardware resource requirements:

| Component | vCPU | RAM | Disk for data storage |
|-----------|------|-----|-----------------------|
| CA | 1 |192 MB | 1 GB |
| Orderer | 2 | 512 MB | 100 GB with the ability to expand |
| Peer | 2 | 2 GB | 50 GB with the ability to expand |
| CouchDB for Peer | 2| 2 GB |50 GB with the ability to expand |

 **Notes:**
 - A vCPU is a virtual core that is assigned to a virtual machine or a physical processor core if the server is not partitioned for virtual machines. You need to consider vCPU requirements when you decide the virtual processor core (VPC) for your deployment in ICP. VPC is a unit of measurement to determine the licensing cost of IBM products. For more information about scenarios to decide VPC, see [Virtual processor core (VPC) ![External link icon](images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SS8JFY_9.2.0/com.ibm.lmt.doc/Inventory/overview/c_virtual_processor_core_licenses.html).
 - These minimum resource levels are sufficient for testing and experimentation. For an environment with a large volume of transactions, it is important to allocate a sufficiently large amount of storage; 250 GB for your peer and 500 GB for your ordering service as an example. The amount of storage to use will depend on the number of transactions and the number of signatures that are required from your network. If you are about to exhaust the storage on your peer or orderer, you must deploy a new peer or orderer with a larger file system and let it sync via your other components on the same channels.

#### Storage considerations
{: #helm-icp-storage-considerations}

* You need to determine the storage that your components will use. If you use the default settings, the Peer Helm chart creates a new Persistent Volume claim with the name of `my-data-pvc` for your peer data. If you select CouchDB as your ledger database, the Helm chart creates another Persistent Volume claim with the name of `statedb-pvc` for your ledger database.
* If you do not want to use the default storage settings, ensure that a *new* storageClass is set up during the ICP installation or the Kubernetes system administrator needs to create a storageClass before you deploy.
* [Dynamic provisioning ![External link icon](/images/external_link.svg "External link icon")]( https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/ "Dynamic Volume Provisioning") is available only for amd64 nodes in ICP. Therefore, if your cluster includes a mix of s390x and amd64 worker nodes, dynamic provisioning cannot be used.
* If dynamic provisioning is not used, [Persistent Volumes ![External link icon](/images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/storage/persistent-volumes/ "Persistent Volumes") must be created and set up with labels that can be used to refine the Kubernetes Persistent Volume Claim (PVC) bind process.
* If you use NFS v2/v3 Persistent Volumes, you must enable the **NFS status monitor for NFSv2/v3 Filesystem Locks** module, which is also known as **rpc-statd**, on the host system where the NFS file system exists. This module allows your NFS file system to check for exclusive locks on files that other processes hold. Run the following commands to enable this module:
  ```
  sudo systemctl enable rpc-statd
  ```
  {:codeblock}

  ```
  sudo systemctl start rpc-statd
  ```
  {:codeblock}

## Installing ICP
{: #icp-setup-install}

Complete the following steps to install and set up {{site.data.keyword.cloud_notm}} Private in your environment.

1. Install an [IBM Cloud Private ![External link icon](images/external_link.svg "External link icon") ](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/kc_welcome_containers.html) cluster at version 3.1.0. If you want to use the Helm chart for development, test, or experimentation, you can install the [ICP Community Edition version 3.1.0 ![External link icon](/images/external_link.svg "External link icon")]( https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/kc_welcome_containers.html "{{site.data.keyword.cloud_notm}} Private-CE version 3.1.0") for free.

2. Install the IBM Cloud Private CLI [3.1.0 ![External link icon](images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/manage_cluster/install_cli.html) to install and operate the CA.

After you install ICP, you can continue to [import the {{site.data.keyword.blockchainfull_notm}} Platform for ICP Helm chart](howto/helm_install_icp.html) into your ICP cluster.
