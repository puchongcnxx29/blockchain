---

copyright:
  years: 2017, 2018
lastupdated: "2018-12-07"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# Deploying peers in {{site.data.keyword.cloud_notm}} Private
{: #peer-icp}


***[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

The following instructions describe how to deploy an {{site.data.keyword.blockchainfull}} Platform peer on {{site.data.keyword.cloud_notm}} Private (ICP). These instructions allow you to connect to an {{site.data.keyword.blockchainfull_notm}} Platform on ICP. If you want to connect a peer to a Starter Plan or Enterprise Plan network on {{site.data.keyword.cloud_notm}}, see [Deploying peers to connect to Starter Plan or Enterprise Plan](peer_deploy_ibp.html).
{:shortdesc}

Before you deploy a peer, review the [Considerations and limitations](../ibp-for-icp-about.html#ibp-icp-considerations).

## Resources required
{: #peer-resources-required}

Ensure that your ICP system meets the minimum hardware resource requirements:

| Component | vCPU | RAM | Disk for data storage |
|-----------|------|-----|-----------------------|
| Peer | 2 | 2 GB | 50 GB with the ability to expand |
| CouchDB for Peer | 2| 2 GB |50 GB with the ability to expand |

 **Notes:**
 - A vCPU is a virtual core that is assigned to a virtual machine or a physical processor core if the server is not partitioned for virtual machines. You need to consider vCPU requirements when you decide the virtual processor core (VPC) for your deployment in ICP. VPC is a unit of measurement to determine the licensing cost of IBM products. For more information about scenarios to decide VPC, see [Virtual processor core (VPC) ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SS8JFY_9.2.0/com.ibm.lmt.doc/Inventory/overview/c_virtual_processor_core_licenses.html).
 - These minimum resource levels are sufficient for testing and experimentation. For an environment with a large volume of transactions, it is important to allocate a sufficiently large amount of storage; 250 GB for your peer as an example. The amount of storage to use will depend on the number of transactions and the number of signatures that are required from your network. If you are about to exhaust the storage on your peer or orderer, you must deploy a new peer or orderer with a larger file system and let it sync via your other components on the same channels.

## Storage
{: #storage}

You need to determine the storage that your peer will use. If you use the default settings, the Helm chart will create a new 8 Gi Persistent Volume Claim (PVC) with the name of `my-data-pvc` for your peer data, and another 8 Gi PVC with the name of `statedb-pvc` for your state database.

If you do not want to use the default storage settings, ensure that a *new* `storageClass` is set up during the ICP installation or the Kubernetes system administrator needs to create a storageClass before you deploy.

You can choose to deploy the peer on either the amd64 or s390x platforms. However, be aware that [Dynamic provisioning](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/) is only available for amd64 nodes in ICP. If your cluster includes a mix of s390x and amd64 worker nodes, dynamic provisioning cannot be used.

If you do not use dynamic provisioning, [Persistent Volumes ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) must be created and setup with labels that can be used to refine the Kubernetes PVC bind process.

## Prerequisites for deploying a peer
{: #prerequisites-peer-icp}

1. Before you can install a peer on ICP, you must [install ICP](../ICP_setup.html) and [install the {{site.data.keyword.blockchainfull_notm}} Platform Helm chart](helm_install_icp.html).

2. If you use the Community Edition and you want to run this Helm chart on an ICP cluster without Internet connectivity, you need to create archives on an Internet-connected machine before you can install the archives on your ICP cluster. For more information, see [Adding featured applications to clusters without Internet connectivity ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/app_center/add_package_offline.html "Adding featured applications to clusters without Internet connectivity"){:new_window}. Note that you can find the specification file `manifest.yaml` under `ibm-blockchain-platform-dev/ibm_cloud_pak` in the Helm chart.

3. You must first [deploy a CA](CA_deploy_icp.html) on ICP. You need to use the CA to create a [peer configuration file and store it as a Kubernetes secret in ICP](#peer-config-file).

4. Retrieve the value of the cluster Proxy IP address of your CA from the ICP console. **Note:** You will need to be a [Cluster administrator ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/user_management/assign_role.html "Cluster administrator roles and actions") to access your proxy IP. Log in to the ICP cluster. In the left navigation panel, click **Platform** and then **Nodes** to view the nodes that are defined in the cluster. Click the node with the role `proxy` and then copy the value of the `Host IP` from the table. **Important:** Save this value and you will use it when you configure the `Proxy IP` field of the Helm chart.


## Creating the peer configuration secret
{: #peer-config-file}

To deploy a peer, you need to create a configuration file that contains important information about the peer identity and your certificate authority. Then, you need to pass this file to the Helm chart during configuration by using a [Kubernetes Secret ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/configuration/secret/) object. This file will allow your peer to get the certificates it needs from the certificate authority to join a blockchain network. It also contains an admin certificate that will allow you to operate your peer. Follow the instructions for [using the CA to deploy an orderer or peer](CA_operate.html#deploy-orderer-peer) before peer configuration.

You need to provide the CSR hostnames to the configuration file. This includes the `service host name` that will be the same value as the `helm release name` that you specify during deployment. For example, if you specify a `helm release name` of `org1peer1`, remember to insert the following value in the `"csr"` section of the file:
```
"csr": {
  "hosts": [
     "9.42.134.44",
    "org1peer1"
  ]
}
```
{:codeblock}

After you save the configuration file, you need to encode it to the base64 format before you provide it to ICP as a secret object. A Kubernetes secret allows you to protect and share information without having to pass it directly to the deployment.

1. To encode the configuration file in the base64 format, run the following commands from a terminal window:
```
export FLAG=$(if [ "$(uname -s)" == "Linux" ]; then echo "-w 0"; else echo "-b 0"; fi)
cat <config_file> | base64 $FLAG
```
{:codeblock}

**Note:** It is important that the string that the above command generates is formatted as a single line. It should look similar to:

```
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tDQpNSUlFbERDQ0EzeWdBd0lCQWdJUUFmMmo2MjdLZGNpSVE0dHlTOCs4a1RBTkJna3Foa2lHOXcwQkFRc0ZBREJoDQpNUXN3Q1FZRFZRUUdFd0pWVXpFVk1CTUdBMVVFQ2hNTVJHbG5hVU5sY25RZ1NXNWpNUmt3RndZRFZRUUxFeEIzDQpkM2N1WkdsbmFXTmxjblF1WTI5dE1TQXdIZ1lEVlFRREV4ZEVhV2RwUTJWeWRDQkhiRzlpWVd3Z1VtOXZkQ0JEDQpRVEFlRncweE16QXpNRGd4TWpBd01EQmFGdzB5TXpBek1EZ3hNakF3TURCYU1FMHhDekFKQmdOVkJBWVRBbFZUDQpNUlV3RXdZRFZRUUtFd3hFYVdkcFEyVnlkQ0JKYm1NeEp6QWxCZ05WQkFNVEhrUnBaMmxEWlhKMElGTklRVElnDQpVMlZqZFhKbElGTmxjblpsY2lC
```

But not like this:

```
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tDQpNSUlFbERDQ0EzeWdBd0lCQWdJUUFmMmo2MjdL
ZGNpSVE0dHlTOCs4a1RBTkJna3Foa2lHOXcwQkFRc0ZBREJoDQpNUXN3Q1FZRFZRUUdFd0pWVXpF
Vk1CTUdBMVVFQ2hNTVJHbG5hVU5sY25RZ1NXNWpNUmt3RndZRFZRUUxFeEIzDQpkM2N1WkdsbmFX
VEFlRncweE16QXpNRGd4TWpBd01EQmFGdzB5TXpBek1EZ3hNakF3TURCYU1FMHhDekFKQmdOVkJB
WVRBbFZUDQpNUlV3RXdZRFZRUUtFd3hFYVdkcFEyVnlkQ0JKYm1NeEp6QWxCZ05WQkFNVEhrUnBa
```

Save the resulting output for Step 4 below.

2. Log in to the ICP console. In the left navigation panel, click **Configuration** and then **Secrets**. Click the **Create Secret** button to open a pop panel that allows you to generate a new secret object.

3. On the **General** tab, complete the following fields:
  - **Name:** Give your secret a unique name within your cluster. You will use this name when deploying your peer. The name must be all lowercase.  
  **Note:** When you deploy a peer, a new secret is automatically generated by the deployment with the name `<helm_release_name>-secret`. Therefore, when you name your secret, be sure the name of the secret differs from the `<helm_release_name>-secret` . Otherwise, the helm chart deployment will fail because the secret it tries to create already exists.
  - **Namespace:** The namespace to add your secret. Select the `namespace` that you want to deploy your peer to.
  - **Type:** Enter the value `Opaque`.

4. Click the **Data** tab on the left navigation pane of this window.
  - In the `Name` fields, specify `secret.json` and in the value field paste in the `base64` encoded string  of your configuration file.

5. (Optional) If you plan to use `CouchDB` as your state database, you need to add two additional values to this secret object. In the **Data** tab, click the **Add data** button to add the CouchDB userid and password to be used for the database when the container is deployed.
  1. Create your user name and password and encode the values in the base64 format. Run the following commands in a terminal window and replace `admin` and `adminpw` with the values that you want to use.
    ```
    echo -n 'couch' | base64 $FLAG
    echo -n 'couchpw' | base64 $FLAG
    ```
    {:code_block}

  2. In the **Name** field, enter the value `couchdbuser`. In the corresponding **Value** field, enter the result of `echo -n 'couch' | base64 $FLAG` from step one above.
  3. Click the **Add data** button to add a second key value pair.
  4. In the second **Name** field, enter the value `couchdbpwd`. In the corresponding **Value** field, enter the result of `echo -n 'couchpw' | base64 $FLAG` from step one above.

6. Click **Create** to save your secret object.

**Note:** The peer configuration secret will not be removed from your ICP cluster when you delete your Helm release. If you delete your peer, you need to delete this secret as well.

## Configuration
{: #peer-configuration}

After you create your peer configuration secret object, you can configure and install your peer in ICP with the following steps. You can install only one peer at a time.


1. Log in to the ICP console and click the **Catalog** link in the upper right corner.
2. Click `Blockchain` in the left navigation panel to locate the tile labeled `ibm-blockchain-platform-prod` or `ibm-blockchain-platform-dev` if you downloaded  the Community edition. Click the tile to open it and you can see a Readme file that includes information about installing and configuring the Helm chart.
3. Click the **Configuration** tab on the top of the panel or click the **Configure** button in the lower right corner.
4. Specify the values for the [General configuration parameters](#peer-global-parameters) and accept the license agreement.
5. Open the `All parameters` twistie and specify the value for the [Global configuration parameters](#peer-global-parameters).
6. Scroll down to the **Peer configuration** section. Select the `Install Peer` check box and complete the [configuration settings](#peer-parameters) for the peer.
7. Click **Install**.

### Configuration parameters
{: #icp-peer-configuration-parms}

The following table lists the configurable parameters of the {{site.data.keyword.blockchainfull_notm}} Platform, **specific to the peer component**, and their default values. If you are experimenting with the peer or getting started for the first time, use the default values of the database and storage parameters. Scroll to the peer component section to check the  `Install Peer` check box and provide the associated configuration information for that component only. **Although the Helm chart UI says no further configuration is required, you need to enter certain parameters to deploy a peer.**

#### General and global configuration parameters
{: #peer-global-parameters}

|  Parameter     | Description    | Default  | Required |
| --------------|-----------------|-------|------- |
| `Helm release name`| The name of your Helm release. Must begin with a lowercase letter and end with any alphanumeric character, must only contain hyphens and lowercase alphanumeric characters. You must use a unique Helm release name each time you attempt to install a component. **Important:** This value must match the value you used to generate the 'service host name' for the "hosts" field in your [JSON secret file.](#peer-config-file) | none | yes |
| `Target namespace`| Choose the Kubernetes namespace to install the Helm chart. | none | yes |
|**Global configuration**| Parameters which apply to all components in the Helm chart|||
| `Service account name`| Enter the name of the [service account ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/) that you will use to run the pod. | default | no |

#### Peer configuration parameters
{: #peer-parameters}

|  Parameter     | Description    | Default  | Required |
| --------------|-----------------|-------|------- |
|**Cluster configuration** |**Cluster configuration information** | ||
| `Install Peer` | Select to install a peer|unchecked | yes, if you want to deploy a peer |
| `Peer worker node architecture`| Select your cloud platform architecture (AMD64 or S390x)| AMD64 | yes |
| `Peer image repository`| Location of the Peer Helm chart. This field is autofilled to the installed path. | ibmcom/ibp-fabric-peer | yes |
| `Peer Docker image tag`|Value of the tag that is associated with the peer image. |1.2.1, autofilled to correct value.|yes|
| `Peer configuration`| You can customize the configuration of the peer by pasting your own `core.yaml` configuration file in this field. To see a sample `core.yaml` file, see [`core.yaml` sample config ![External link icon](../images/external_link.svg "External link icon")](https://github.com/hyperledger/fabric/blob/release-1.2/sampleconfig/core.yaml) **For advanced users only**. | none | no |
| `Peer configuration secret (Required)`|Name of the [Peer configuration secret](#peer-config-secret) you created in ICP. | none | yes |
|`Organization MSP (Required)`| You can create a new Organization MSPID value such as 'org1' or specify an existing Organization MSP that the peer will be part of. If you have deployed an orderer organization, ensure that any peer MSPIDs are different from your orderer MSPID. Also, make a note of this value, as you will need it for your `CORE_PEER_LOCALMSPID` and `configtx.yaml` later. | none | yes |
|`Peer service type`| Used to specify whether [external ports should be exposed ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types) on the peer. Select NodePort to expose the ports externally (recommended), and ClusterIP to not expose the ports. LoadBalancer and ExternalName are not supported in this release. | NodePort | yes |
| `State database`| The [state database](../glossary.html#state-database) used to store your channel ledger. The peer needs to use the same database as your [blockchain network](../v10_dashboard.html#network-preferences). | none | yes |
|`CouchDB image repository`| Applies only if CouchDB was selected as the ledger database. This field is autofilled to the installed path. If you are using the Community Edition and don't have internet access, it should match the directory where you downloaded the Fabric CouchDB image.| ibmcom/ibp-fabric-couchdb | yes |
| `CouchDB Docker image tag`| Applies only if CouchDB was selected as the ledger database. Value of the tag associated with the CouchDB image. | Autofilled to correct value. | yes |
| `Peer Data persistence enabled`| Enable the ability to persist data after cluster restarts or fails. See [storage in Kubernetes ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/storage/ "Volumes") for more information. *If unchecked, all data will be lost in the event of a failover or pod restart.* | checked | no |
| `Peer use dynamic provisioning`| Check to enable dynamic provisioning for storage volumes. | checked | no |
| `Peer persistent volume claim`| For new claim only. Enter a name for your new Persistent Volume Claim (PVC) to be created. | my-data-pvc | no |
| `Peer storage class name`| Specify a storage class name for the peer. | Blank if creating a new PVC; otherwise, specify the storage class that is associated with the existing PVC. | no |
| `Peer existing volume claim`| Specify the name of an existing Volume Claim and leave all other fields blank. | none | no |
| `Peer selector label`| [Selector label ![External link icon](../images/external_link.svg "External link icon") ](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/ "Labels and Selectors") for your PVC.| none | no |
| `Peer selector value`| [Selector value ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/ "Labels and Selectors") for your PVC. | none | no |
| `Peer storage access mode`| Specify the storage [access mode ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes "Access Modes") for the PVC.  | ReadWriteMany| no |
| `Peer volume claim size`| Size of the Volume Claim, must be larger than 2Gi. | 8Gi  | yes |
| `State database persistent volume claim`| For new claim only. Enter a name for your new Persistent Volume Claim (PVC) to be created. | statedb-pvc | no |
| `State database storage class name`| Specify a storage class name for state database. | none | no |
| `State database existing volume claim`| Specify the name of an existing Volume Claim and leave all other fields blank. | none | no |
| `State database selector label`| [Selector label](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) for your PVC| none | no |
| `State database selector value`| [Selector value](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) for your PVC | none | no |
| `State database storage access mode`| Specify the storage [access mode ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes "Access Modes") for the PVC. | ReadWriteMany| no |
| `State database volume claim size`| Choose the size of disk to use. | 8Gi | yes |
| `CouchDB - Data persistence enabled`| For CouchDB container, ledger data will be available when the container restarts. *If unchecked, all data will be lost in the event of a failover or pod restart.*| checked | no |
| `CouchDB - Use dynamic provisioning`| For CouchDB container use Kubernetes dynamic storage.| checked | no |
| `Peer CPU request` | Minimum number of CPUs to allocate to the peer. | 1 | yes |
| `Peer CPU limit` | Maximum number of CPUs to allocate to the peer.| 2 | yes |
| `Peer Memory request` | Minimum amount of memory to allocate to the peer. | 1Gi | yes |
| `Peer Memory limit` | Maximum amount of memory to allocate to the peer. | 4Gi | yes |
| `CouchDB CPU request` | Minimum number of CPUs to allocate to CouchDB.| 1 | yes |
| `CouchDB CPU limit` | Maximum number of CPUs to allocate to CouchDB. | 2 | yes |
| `CouchDB Memory request` | Minimum amount of memory to allocate to CouchDB.| 1Gi | yes |
| `CouchDB Memory limit` | Maximum amount of memory to allocate to CouchDB. | 4Gi | yes |


When CouchDB is selected as the peer database, two containers are created in the pod, one for the peer and one for CouchDB.
The Peer container includes a single volume mount to the Peer PVC that stores the blocks and transactions on the file system. The CouchDB container mounts the peer state database PVC that contains the ledger data.

<!-- LevelDB is not supported on the Peer
When LevelDB is selected as the peer database, a single container is created in the pod for running both the peer and LevelDB
processes. This container has two volume mounts, one for the Peer PVC and the second volume mount is for the peer state database PVC that contains the ledger data.
-->
<!--
| Peer database selection  | Contents of Container #1  | Contents of Container #2 |
| --------------|-----------------|---------------|
| CouchDB | Peer that mounts the Peer PVC| CouchDB that mounts the state database PVC |
| LevelDB | Peer and LevelDB that mount the Peer PVC and the state database PVC | n/a |
-->

### Using the Helm command line to install the Helm release
{: #icp-helm-cli}

Alternatively, you can use the `helm` CLI to install the Helm release. Before you run the `helm install` command, ensure that you [add your cluster's Helm repository to the Helm CLI environment ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/app_center/add_int_helm_repo_to_cli.html "Adding the internal Helm repository to Helm CLI").

You can set the parameters required for installation by creating a `yaml` file and passing it to the following `helm install` command.
```
helm install --name <helm_release_name>  <helm_chart> \
--version <helm_chart_version> \
--values <customvalues.yaml> \
--tls
```
{:codeblock}

Where:
- `<helm_release name>` represents the name you want to give your helm release.
- `<helm_chart>` represents the name of the Helm chart that is imported into the catalog.
- `<helm_chart_version>` represents the version of the Helm chart that is imported into the catalog.
- `<customvalues.yaml>` is the name of the yaml file containing the configuration parameters.

For example:
```
helm install --name jnchart2 mycluster/ibm-blockchain-platform \
--version 1.1.0 \
--values peer-s390x-values.yaml \
--tls
```

You can create a new `yaml` file by editing `values.yaml` included in the downloaded archive file, which includes all the necessary parameters with their default settings.

## Verify the peer installation
{: #verify-installation-icp}

After you complete the configuration parameters and click the **Install** button, click the **View Helm Release** button to view your deployment. If it was successful, you should see the value 1 in the `DESIRED`, `CURRENT`, `UP TO DATE`, and `AVAILABLE` fields in the Deployment table. You may need to click refresh and wait for the table to be updated. You can also find the Deployment table by clicking the **Menu** icon in the upper left corner In the ICP console. From the menu list, click **Workloads** and then **Helm Releases**.

If you scroll down to the `Notes` section, there is important information that you will use when [operating your peer](peer_operate_icp.html).

## Viewing the peer logs
{: #peer-deploy-view-logs}

Peer logs can be viewed by using [kubectl CLI commands](peer_operate_icp.html#peer-kubectl-configure) or through [Kibana ![External link icon](../images/external_link.svg "External link icon")](https://www.elastic.co/products/kibana "Your window into the Elastic Search"). For more information, see these [instructions for accessing the logs](peer_operate_icp.html#peer-icp-view-logs).

## What's next

After you deploy the peer, you need to complete several operational steps before you can submit transactions to and read the distributed ledger from the blockchain network. For more information, see [Operating peers with a multi-cloud network](peer_operate_icp.html).
