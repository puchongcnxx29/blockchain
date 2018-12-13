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

# Deploying an orderer in {{site.data.keyword.cloud_notm}} Private
{: #orderer-deploy}

***[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

Orderers authenticate clients, order transactions, and broadcast transactions in a blockchain network with the orderer component. For more information about orderers and the role that they play in a blockchain network, see [Overview on blockchain components](../blockchain_component_overview.html).
{:shortdesc}

Before you a deploy an ordering service, review the [Considerations and limitations](../ibp-for-icp-about.html#ibp-icp-considerations).

## Resources required
{: #ca-resources-required}

Ensure that your ICP system meets the minimum hardware resource requirements:

| Component | vCPU | RAM | Disk for data storage |
|-----------|------|-----|-----------------------|
| Orderer | 2 | 512 MB | 100 GB with the ability to expand |


 **Notes:**
 - A vCPU is a virtual core that is assigned to a virtual machine or a physical processor core if the server is not partitioned for virtual machines. You need to consider vCPU requirements when you decide the virtual processor core (VPC) for your deployment in ICP. VPC is a unit of measurement to determine the licensing cost of IBM products. For more information about scenarios to decide VPC, see [Virtual processor core (VPC) ![External link icon](images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SS8JFY_9.2.0/com.ibm.lmt.doc/Inventory/overview/c_virtual_processor_core_licenses.html).
 - These minimum resource levels are sufficient for testing and experimentation. For an environment with a large volume of transactions, it is important to allocate a sufficiently large amount of storage; 500 GB for your ordering service as an example. The amount of storage to use will depend on the number of transactions and the number of signatures that are required from your network. If you are about to exhaust the storage on your orderer, you must deploy a new orderer with a larger file system and let it sync via your other components on the same channels.

## Storage
{: #storage}

You need to determine the storage that your orderer will use. If you use the default settings, the Helm chart will create a new 8 Gi Persistent Volume Claim (PVC) with the name of `my-data-pvc` for your orderer data, and another 8 Gi PVC with the name of `statedb-pvc` for your state database.

If you do not want to use the default storage settings, ensure that a *new* `storageClass` is set up during the ICP installation or the Kubernetes system administrator needs to create a storageClass before you deploy.

You can choose to deploy the orderer on either the amd64 or s390x platforms. However, be aware that [Dynamic provisioning](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/) is only available for amd64 nodes in ICP. If your cluster includes a mix of s390x and amd64 worker nodes, dynamic provisioning cannot be used.

If you do not use dynamic provisioning, [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) must be created and setup with labels that can be used to refine the Kubernetes PVC bind process.


## Prerequisites for deploying an orderer
{: #prerequisites-orderer-icp}

1. Before you can install an orderer on ICP, you must [install ICP](../ICP_setup.html) and [install the {{site.data.keyword.blockchainfull_notm}} Platform Helm chart](helm_install_icp.html).

2. If you use the Community Edition and you want to run this Helm chart on an ICP cluster without Internet connectivity, you need to create archives on an Internet-connected machine before you can install the archives on your the ICP cluster. For more information, see [Adding featured applications to clusters without Internet connectivity ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/app_center/add_package_offline.html "Adding featured applications to clusters without Internet connectivity"){:new_window}. Note that you can find the specification file `manifest.yaml` under ibm-blockchain-platform-dev/ibm_cloud_pak in the Helm chart.

3. Retrieve the value of the cluster Proxy IP address of your CA from the ICP console. **Note:** You will need to be a [Cluster administrator ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/user_management/assign_role.html "Cluster administrator roles and actions") to access your proxy IP. Log in to the ICP cluster. In the left navigation panel, click **Platform** and then **Nodes** to view the nodes that are defined in the cluster. Click the node with the role `proxy` and then copy the value of the `Host IP` from the table. **Important:** Save this value and you will use it when you configure the `Proxy IP` field of the Helm chart.

4. Create an [orderer configuration file and store it as a Kubernetes secret in ICP](#orderer-config-file).

## Creating an orderer configuration file
{: #orderer-config-file}

Before you deploy an orderer, you need to create a configuration file containing important information about the orderer identity and your CA. Then, you need to pass this file to the Helm chart during configuration by using a [Kubernetes Secret ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/configuration/secret/) object. This file will allow the orderer to get the certificates that it needs from the CA to join a blockchain network. It also contains an admin certificate that will allow you to operate the orderer as an admin user. Follow the instructions on [using the CA to deploy an orderer or peer](CA_operate.html#deploy-orderer-peer) before you configure the orderer.

You need to provide the CSR hostnames to the configuration file. This includes the `service host name` that will be based on the `helm release name` that you specify during deployment. The `service host name` is the `helm_release_name` that you specify with the string `-orderer` added at the end. For example, if you specify a `helm release name` of `orderer1`, you can insert the following value in the `"csr"` section of the file:

```
"csr": {
  "hosts": [
    "9.42.134.44",
    "orderer1-orderer"
  ]
}
```

After you save the configuration file, you need to encode it in the base64 format before you provide the file to ICP as a secret object. A Kubernetes secret allows you to protect and share information without having to pass it directly to the deployment.

1. Encode the configuration file in the base64 format by running the following commands from a terminal window:

  ```
  export FLAG=$(if [ "$(uname -s)" == "Linux" ]; then echo "-w 0"; else echo "-b 0"; fi)
  cat <config_file> | base64 $FLAG
  ```
  {:codeblock}

  **Note:** It is important that the string generated by using the command above is formatted as a single line. It should look similar to:

   ```
   LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tDQpNSUlFbERDQ0EzeWdBd0lCQWdJUUFmMmo2MjdLZGNpSVE0dHlTOCs4a1RBTkJna3Foa2lHOXcwQkFRc0ZBREJoDQpNUXN3Q1FZRFZRUUdFd0pWVXpFVk1CTUdBMVVFQ2hNTVJHbG5hVU5sY25RZ1NXNWpNUmt3RndZRFZRUUxFeEIzDQpkM2N1WkdsbmFXTmxjblF1WTI5dE1TQXdIZ1lEVlFRREV4ZEVhV2RwUTJWeWRDQkhiRzlpWVd3Z1VtOXZkQ0JEDQpRVEFlRncweE16QXpNRGd4TWpBd01EQmFGdzB5TXpBek1EZ3hNakF3TURCYU1FMHhDekFKQmdOVkJBWVRBbFZUDQpNUlV3RXdZRFZRUUtFd3hFYVdkcFEyVnlkQ0JKYm1NeEp6QWxCZ05WQkFNVEhrUnBaMmxEWlhKMElGTklRVElnDQpVMlZqZFhKbElGTmxjblpsY2lC
   ```
   not like this:

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
  - **Name:** Give your secret a unique name within your cluster. You will use this name when deploying your orderer. The name must be all lower case.  
  **Note:** When you deploy an orderer, a new secret is automatically generated by the deployment with the name `<helm_release_name>-orderer-mspsecret`. Therefore, when you name your secret, be sure the name of the secret differs from `<helm_release_name>-orderer-mspsecret`. Otherwise, the helm chart deployment will fail because the secret it tries to create already exists.
  - **Namespace:** The namespace to add your secret. Select the `namespace` that you want to deploy your orderer to.
  - **Type:** Enter the value `Opaque`.

4. Click the **Data** tab on the left navigation pane of this window. In the `Name` fields, specify `secret.json` and in the value field paste in the `base64` encoded string of your configuration file.

5. Click **Create** to save your secret object.

**Note:** The orderer configuration secret will not be removed from your ICP cluster when you delete your Helm release. If you delete your orderer, you need to delete this secret too.

## Configuration
{: #orderer-configuration}

After you create your orderer configuration secret object, you can configure and install your orderer in ICP with the following steps. You can install only one orderer at a time.

1. Log in to the ICP console and click the **Catalog** link in the upper right corner.
2. Click `Blockchain` in the left navigation panel to locate the tile labelled `ibm-blockchain-platform-prod` or `ibm-blockchain-platform-dev` if you downloaded the Community edition. Click the tile to open it and you can see a Readme file that includes information about installing and configuring the Helm chart.
3. Click the **Configuration** tab on the top of the panel or click the **Configure** button in the lower right corner.
4. Specify the values for the [General configuration parameters](#orderer-global-parameters) and accept the license agreement.
5. Open the `All parameters` twistie and specify the value for the [Global configuration parameters](#orderer-global-parameters).
6. Scroll down to the **Orderer configuration** section. Select the `Install Orderer` check box and complete the [configuration settings](#orderer-parameters) for the orderer.
7. Click **Install**.


**Note:** when deploying the orderer to s390x, the init container might return the following error:

`panic: Error while trying to open DB: no locks available`

This is because the orderer uses a flat-file DB and the NFS Filesystem requires an additional package to allow the ability for the ordering service to query files for checking if there are exclusive locks on them, and for creating exclusive locks. To fix the issue, you must enable the `rpc-statd` package.

`sudo systemctl enable rpc-statd`
`sudo systemctl start rpc-statd`

This must be done on the system's where the NFS filesystem is served from.

### Configuration parameters
{: #icp-orderer-configuration-parms}

The following table lists the configurable parameters of the {{site.data.keyword.blockchainfull_notm}} Platform, **specific to the orderer component**, and their default values. **Although the Helm chart UI says no further configuration is required, you need to enter certain parameters to deploy an orderer.**

#### General and global configuration parameters
{: #orderer-global-parameters}

|  Parameter     | Description    | Default  | Required |
| --------------|-----------------|-------|------- |
| `Helm release name`| The name of your helm release. Must begin with a lowercase letter and end with any alphanumeric character, must only contain hyphens and lowercase alphanumeric characters. You must use a unique Helm release name each time you attempt to install a component. **Important:** This value must match the value that you used to generate the 'service host name' for the "hosts" field in your [JSON secret file.](#orderer-config-file) | none | yes  |
| `Target namespace`| Choose the Kubernetes namespace to install the Helm chart. | none | yes |
|**Global configuration**| Parameters which apply to all components in the Helm chart|||
| `Service account name`| Enter the name of the [service account ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/) that you will use to run the pod. | default | no |

#### Orderer configuration parameters
{: #orderer-parameters}

|  Parameter     | Description    | Default  | Required |
| --------------|-----------------|-------|------- |
| `Install Orderer`| Select to install an orderer. | unchecked | yes, if you want to deploy an orderer |
| `Orderer worker node architecture`| Select your ICP worker node architecture (AMD64 or S390X). | Autodetected architecture based on your master node | yes |
| `Orderer configuration`| You can customize the configuration of the orderer by pasting your own `orderer.yaml` configuration file in this field. To see a sample `orderer.yaml` file, see [`orderer.yaml` sample config ![External link icon](../images/external_link.svg "External link icon")](https://github.com/hyperledger/fabric/blob/release-1.2/sampleconfig/orderer.yaml) **For advanced users only**. | none | no |
| `Organization MSP secret (Required)`| Specify the name of the secret object that contains organization MSP certificates and keys. | none | yes |
| `Orderer data persistence enabled` | Data will be available when the container restarts. If unchecked, all data will be lost in the event of a failover or pod restart. | checked | no |
| `Orderer use dynamic provisioning` | Check to enable dynamic provisioning for storage volumes. | checked | no |
| `Orderer image repository` | Location of the Orderer Helm chart. This field is autofilled to the installed path. If you are using the Community Edition and don't have internet access, change this field to the location where you downloaded the Fabric orderer image. | ibmcom/ibp-fabric-orderer | no |
| `Orderer Docker image tag`| A record of the Docker image. This field is autofilled to the image version. Do not change it.| 1.2.1 | yes |
| `Orderer consensus type`| The consensus type of the ordering service. | SOLO | yes |
| `Orderer organization name`| Specify the name you would like to use for the orderer organization. If you also plan to deploy peers, make sure to use a different name than the one you will give to your peers. For example, give your orderer org a name like `ordererOrg` | none | yes |
| `Orderer Org MSP ID`| Specify the name you want to use for the MSP ID of the orderer organization. This should be the same name you give to your orderer organization, and will be set as an environment variable by the deployment process. Make a note of this value and you will need to reference it later. | none | yes |
| `Orderer storage class name`| Specify a storage class name for the orderer. | none | Depends on how the ICP cluster is configured. Check with your cluster administrator |
| `Orderer existing volume claim`| Specify the name of an existing Volume Claim and leave all other fields blank. | none | no |
| `Orderer selector label`| [Selector label ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) for your PVC. | none | no |
| `Orderer selector value`| [Selector value ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) for your PVC. | none | no |
| `Orderer storage access mode`| Specify the storage [access mode ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes) for the PVC. | ReadWriteMany | yes |
| `Orderer volume claim size`| Choose the size of disk to use, which must be at least 2 Gi. | 8 Gi | yes |
| Orderer service type` | Used to specify whether [external ports should be exposed ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types) on the peer. Select NodePort to expose the ports externally (recommended), and ClusterIP to not expose the ports. LoadBalancer and ExternalName are not supported in this release | NodePort | yes |
| `Orderer CPU request`| Specify the minimum number of CPUs to allocate to the Orderer. | 1 | yes |
| `Orderer CPU limit`| Specify the maximum number of CPUs to allocate to the Orderer. | 2 | yes |
| `Orderer memory request`| Specify the minimum amount of memory to allocate to the Orderer. | 1Gi | yes |
| `Orderer memory limit`| Specify the maximum amount of memory to allocate to the Orderer. | 2Gi | yes |

### Using the Helm command line to install the Helm release
{: #icp-helm-cli}

Alternatively, you can use the Helm CLI to install the Helm release. Before you run the `helm install` command, ensure that you [add your cluster's Helm repository to the Helm CLI environment ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/app_center/add_int_helm_repo_to_cli.html "Adding the internal Helm repository to Helm CLI").

You can set the parameters required for installation by creating a `yaml` file and passing it to the following `helm install` command.

```
helm install --name <helm_release_name>  <helm_chart> \
--version <helm_chart_version> \
--values <customvalues.yaml> \
--tls
```
{:codeblock}

where:

- `<helm_release name>` represents the name you want to give your helm release.
- `<helm_chart>` represents the name of the Helm chart that is imported into the catalog.
- `<helm_chart_version>` represents the version of the Helm chart that is imported into the catalog.
- `<customvalues.yaml>` is the name of the yaml file containing the configuration parameters.

For example:

```
helm install --name jnchart2 mycluster/ibm-blockchain-platform \
--version 1.1.0 \
--values orderer-s390x-values.yaml \
--tls
```

You can create a new `yaml` file by editing `values.yaml` included in the downloaded archive file, which includes all the necessary parameters with their default settings.

## Verifying the orderer installation

After you complete the configuration parameters and click the **Install** button, click the **View Helm Release** button to view your deployment. If it was successful, you should see the value 1 in the `DESIRED`, `CURRENT`, `UP TO DATE`, and `AVAILABLE` fields in the Deployment table. You may need to click refresh and wait for the table to be updated. You can also find the Deployment table by clicking the **Menu** icon in the upper left corner In the ICP console. From the menu list, click **Workloads** and then **Helm Releases**.

## Viewing the orderer logs
{: #orderer-view-logs}

Component logs can be viewed from the command line by using the [`kubectl CLI commands`](orderer_operate.html#orderer-kubectl-configure) or through [Kibana ![External link icon](../images/external_link.svg "External link icon")](https://www.elastic.co/products/kibana "Your window into the Elastic Search"), which is included in your ICP cluster. For more information, see these [instructions for accessing the logs](orderer_operate.html#orderer-view-logs).
