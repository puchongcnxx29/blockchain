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

# Deploying a Certificate Authority on {{site.data.keyword.cloud_notm}} Private
{: #CA-deploy}

***[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

After you import the {{site.data.keyword.blockchainfull}} Platform on {{site.data.keyword.cloud_notm}} Private (ICP) Helm chart, you can deploy the individual components. The Certificate Authority (CA) is the root of trust for your organization and allows you to generate credentials for the other components you will deploy. As a result, you need to deploy a CA before you deploy the other components. Each organization in a multi-cloud blockchain network must deploy its own CA.  For more information about CAs and the role that they play in a blockchain network, see [Certificate Authorities](../blockchain_component_overview.html#ca).
{:shortdesc}

Before you a deploy a Certificate Authority, review the [Considerations and limitations](../ibp-for-icp-about.html#ibp-icp-considerations).

## Resources required
{: #ca-resources-required}

Ensure that your ICP cluster meets the minimum hardware resource requirements:

| Component | vCPU | RAM | Disk for data storage |
|-----------|------|-----|-----------------------|
| CA | 1 |192 MB | 1 GB |

**Notes:**
- A vCPU is a virtual core that is assigned to a virtual machine or a physical processor core if the server is not partitioned for virtual machines. You need to consider vCPU requirements when you decide the virtual processor core (VPC) for your deployment in ICP. VPC is a unit of measurement to determine the licensing cost of IBM products. For more information about scenarios to decide VPC, see [Virtual processor core (VPC) ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SS8JFY_9.2.0/com.ibm.lmt.doc/Inventory/overview/c_virtual_processor_core_licenses.html "Virtual Processor Core").
- The data storage requirement depends on how many identities and certificates are stored. CA storage is not as heavy as peer or orderer storage, but this will depend on the use case. The more users, the more storage is needed.
- These minimum resource levels are sufficient for testing and experimentation. For an environment with a large number of transactions, it is important to allocate a sufficiently large amount of storage for your CA. The amount of storage to use will depend on the number of transactions and the number of signatures that are required from your network. If you exhaust the storage on your CA, you must deploy a new CA with a larger file system and let it sync via your other CAs on the same channels.

## Storage
{: #storage}

You need to determine the storage that your CA will use. If you use the default settings, the Helm chart will create a new 2 Gi Persistent Volume Claim (PVC) with the name of `fabric-ca-pvc` for your CA.

If you do not want to use the default storage settings, ensure that a *new* `storageClass` is set up during the ICP installation or the Kubernetes system administrator needs to create a storageClass before you deploy the CA.

You can choose to deploy the CA on either the AMD64 or S390X platforms. However, be aware that [Dynamic provisioning](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/) is available for only AMD64 nodes in ICP. If your cluster includes a mix of S390X and AMD64 worker nodes, dynamic provisioning cannot be used.

If you do not use dynamic provisioning, [Persistent Volumes ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) must be created and set up with labels that can be used to refine the Kubernetes PVC bind process.

## Prerequisites for deploying a CA
{: #prerequisites-ca-icp}

1. Before you can install a CA on ICP, you must [install ICP](../ICP_setup.html) and [install the {{site.data.keyword.blockchainfull_notm}} Platform Helm chart](helm_install_icp.html).

2. If you use the Community Edition and you want to run this Helm chart on an ICP cluster without Internet connectivity, you need to create archives on an Internet-connected machine before you can install the archives on your the ICP cluster. For more information, see [Adding featured applications to clusters without Internet connectivity ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/app_center/add_package_offline.html "Adding featured applications to clusters without Internet connectivity"){:new_window}. Note that you can find the specification file manifest.yaml under ibm-blockchain-platform-dev/ibm_cloud_pak in the Helm chart.

3. Retrieve the value of the cluster Proxy IP address from the ICP console. **Note:** You will need to be a [Cluster administrator ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/user_management/assign_role.html "Cluster administrator roles and actions") to access your proxy IP. Log in to the ICP cluster. In the left navigation panel, click **Platform** and then **Nodes** to view the nodes that are defined in the cluster. Click the node with the role `proxy` and then copy the value of the `Host IP` from the table. **Important:** Save this value and you will use it when you configure the `Proxy IP` field of the Helm chart.

4. Create the CA admin user name and password and store them inside a secret object in ICP. You can find the steps to create the secret in the in the [next section](#admin-secret).

## Creating CA admin Secret
{: #admin-secret}

When your CA first starts, it contains an admin identity that you create to operate your CA. You need to create a user name and password for this admin identity before you deploy the CA. It is important to make a note of these values. They will be used later when you operate your CA and deploy additional network components. Create a [Kubernetes Secret ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/configuration/secret/ "Secrets") to store the admin `username` and `password` for your CA deployment. A Kubernetes secret allows you to protect and share information without having to pass it directly to the deployment.

1. Create an admin user name and password and encode the values in the base64 format. Run the following commands in a terminal window and replace the value of `admin` and `adminpw` with the values that you want to use.
  ```
  echo -n 'admin' | base64
  echo -n 'adminpw' | base64
  ```
  {:code_block}

  **Important:** Save the values of the `admin` and `adminpw` that are created above. Also save the encoded values of these fields, which are required when you create the Kubernetes secret object.

2. Log in to the ICP console. In the left navigation panel, click **Configuration** and then **Secrets**. Click the **Create Secret** button to open a pop panel that allows you to generate a new secret object.

3. On the **General** tab, complete the following fields:
  - **Name:** Give your secret a unique name within your cluster. You will use this name for configuring your CA. The name must be all lower case.
  - **Namespace:** The name space to add your secret. Select the `namespace` that you want to deploy your CA to.
  - **Type:** Enter the value `Opaque`.

4. Leave the **Annotations** tab blank.

5. In the **Data** tab, add the user name and password as key value pairs.
  1. In the first **Name** field, enter `ca-admin-name`.
  2. In the first **Value** field, enter the result of `echo -n admin | base64` from step one above.
  3. Click the **Add data** button to add a second key value pair.
  4. In the second **Name** field, enter `ca-admin-password`.
  5. In the second **Value** field, enter the result of `echo -n adminpw | base64` from step one above.  
    **Figure 1** shows what the ICP console secret might contain depending on the values you specified for the `ca-admin-name` and `ca-admin-password` keys.

    ![ICP console secret](../images/CreateCASecret.png "ICP console secret")  
    *Figure 1. ICP console secret*
  6. Click **Create** to form a new Secret object.

The CA Helm chart requires that you use the names `ca-admin-name` and `ca-admin-password` as the key values to deploy the secret to the Helm release.

**Note:** The CA admin secret is not removed from your ICP cluster when you delete your Helm release. You are responsible for managing your secrets in your ICP cluster. If you plan to deploy another CA in the future, you may reuse the CA admin secret. Otherwise, you are responsible for deleting it from your ICP cluster.

## Configuration
{: #ca-configuration}

After you create your CA admin secret, you can use the following steps to configure and install your CA. You can install only one CA at a time.

1. Log in to the ICP console and click the **Catalog** link in the upper right corner.
2. Click `Blockchain` in the left navigation panel to locate the tile labelled `ibm-blockchain-platform-prod` or `ibm-blockchain-platform-dev` if you downloaded the Community edition. Click the tile to open it and you can see a Readme file that includes information about installing and configuring the Helm chart.
3. Click the **Configuration** tab on the top of the panel or click the **Configure** button in the lower right corner.
4. Specify the values for the [General configuration parameters](#ca-global-parameters) and accept the license agreement.
5. Open the `All parameters` twistie and specify the value for the [Global configuration parameters](#ca-global-parameters).
6. Scroll down to the **CA configuration** section. Select the `Install CA` check box and complete the [configuration settings](#ca-parameters) for the CA.  
7. Click **Install**.

### Configuration parameters
{: #icp-ca-configuration-parms}

The following table lists the configurable parameters of the {{site.data.keyword.blockchainfull_notm}} Platform, **specific to the CA component**, and their default values.  

**Although the Helm chart UI says no further configuration is required, you need to enter certain parameters to deploy a CA.**

#### General and global configuration parameters
{: #ca-global-parameters}

|  Parameter     | Description    | Default  | Required |
| --------------|-----------------|-------|------- |
| `Helm release name`| The name of your Helm release. Must begin with a lowercase letter and end with any alphanumeric character, must only contain hyphens and lowercase alphanumeric characters. You must use a unique Helm release name each time you attempt to install a component. | none | yes |
| `Target namespace`| Choose the Kubernetes namespace to install the Helm chart. | none | yes |
|**Global configuration**| Parameters which apply to all components in the Helm chart|||
| `Service account name`| Enter the name of the [service account ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/ "Configure Service Accounts for Pods") that you will use to run the pod. | default | no |

#### CA configuration parameters
{: #ca-parameters}

|  Parameter     | Description    | Default  | Required |
| --------------|-----------------|-------|------- |
| `Install CA`| Select to install a CA. | unchecked | yes, if you want to deploy a CA |
| `CA name`| Specify a name to use for the Certificate Authority. **Important:** Make a note of this value. It is required later when you configure an orderer or peer. | SampleOrgCA | yes |
| `CA worker node architecture`| Select your ICP worker node architecture (ADM64 or S390X). | AMD64 | yes|
| `CA database type`| The type of database to store CA data. Only SQLite is supported. | SQLite | yes |
| `CA data persistence enabled` | If checked, data will be available when the container restarts. Otherwise, all data will be lost in the event of a failover or pod restart. | checked | no |
| `CA use dynamic provisioning` | Check to enable dynamic provisioning for storage volumes. | checked | no |
| `CA storage class name`| Specify a unique storage class name. Otherwise, the default storage class in the cluster is used. | none | Depends on how the ICP cluster is configured. Check with your cluster administrator |
| `CA existing volume claim`| Specify the name of an existing Volume Claim and leave all other fields blank. | none | no |
| `CA selector label`| [Selector label ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/ "Labels and Selectors") for your PVC. | none | no |
| `CA selector value`| [Selector value ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/ "Labels and Selectors") for your PVC. | none | no |
| `CA storage access mode`| Specify the storage [access mode](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes "Access Modes") for the PVC. | ReadWriteMany | yes |
| `CA volume claim size`| Choose the size of disk to use. | 2Gi | yes |
| `CA image repository`| Location of the CA Helm chart. | ibmcom/ibp-fabric-ca | yes |
| `CA Docker image tag`| Value of the tag that is associated with the CA image. This field is autofilled to the image version. Do not change it.| 1.2.1 | yes |
| `CA service type` | Used to specify whether [external ports should be exposed ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types) on the peer. Select NodePort to expose the ports externally (recommended), and ClusterIP to not expose the ports. LoadBalancer and ExternalName are not supported in this release | NodePort | yes |
| `CA secret (Required)`| Enter the name of the Kubernetes secret object that you created for your `ca-admin-name` and `ca-admin-password`. | none | yes |
| `CA CPU request`| Specify the minimum number of CPUs to allocate to the CA. | 1 | yes |
| `CA CPU limit`| Specify the maximum number of CPUs to allocate to the CA. | 2 | yes |
| `CA memory request`| Specify the minimum amount of memory to allocate to the CA. | 1Gi | yes |
| `CA memory limit`| Specify the maximum amount of memory to allocate to the CA. | 4Gi | yes |
| `CA TLS instance name`| Specify a name of the CA TLS instance that will be used to enroll an orderer or peer. | tlsca | yes |
| `CSR common name`| Specify the Common Name (CN) that the generated CA root cert will present when contacted. | tlsca-common | yes |
| `Proxy IP`| Enter the [Proxy Node IP for the cluster ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/installing/install_proxy.html "IBM Cloud Private installation behind an HTTP proxy") where the CA is deployed. | 127.0.0.1 | yes |


### Using the Helm command line to install the Helm release
{: #icp-helm-cli}

Alternatively, you can use the Helm CLI to install the Helm release. Before you run the `helm install` command, ensure that you [add your cluster's Helm repository to the Helm CLI environment ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/app_center/add_int_helm_repo_to_cli.html "Adding the internal Helm repository to Helm CLI").

You can set the parameters that are required for installation by creating a `yaml` file and passing it to the following `helm install` command.

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
--values ca-s390x-values.yaml \
--tls
```

You can create a new `yaml` file by editing `values.yaml` that is included in the downloaded archive file. The `values.yaml` file includes all the necessary parameters with their default settings.

## Verifying the CA installation
{: #verifiying-ca-installation}

After you complete the configuration parameters and click the **Install** button, click the **View Helm Release** button to view your deployment. If it was successful, you should see the value 1 in the `DESIRED`, `CURRENT`, `UP TO DATE`, and `AVAILABLE` fields in the Deployment table. You may need to click refresh and wait for the table to be updated. You can also find the Deployment table by clicking the **Menu** icon in the upper left corner In the ICP console. From the menu list, click **Workloads** and then **Helm Releases**.

If you scroll down to the `Notes` section, you can find important information that you will use to [operate your CA](CA_operate.html#ca-operate).

After you install {{site.data.keyword.blockchainfull_notm}} Platform CA in ICP, a configmap is created with default environment variables settings. You can then change or add environment variables for the CA server to configure its behavior. For more information about CA server configuration parameters, see [Fabric CA server documentation ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#fabric-ca-server "Fabric CA Server").

After you configure the configmap, you need to restart the CA server before the changes take effect. To restart the CA server, you can delete the Fabric CA server POD. ICP will create a new POD that reflects the changes.

## Viewing the CA logs
{: #ca-deploy-view-logs}

Component logs can be viewed from the command line by using the [`kubectl CLI commands`](CA_operate.html#ca-kubectl-configure) or through [Kibana ![External link icon](../images/external_link.svg "External link icon")](https://www.elastic.co/products/kibana "Your window into the Elastic Search"), which is included in your ICP cluster. For more information, see these [instructions for accessing the logs](CA_operate.html#ca-operate-view-logs).

## Operating your CA
{: #operate-ca}

Your CA will be the root of trust for your organization. You need to use your CA to generate certificates for your other components.  As a result, before deploying an orderer or a peer, you must [set up your CA and complete several operational steps](CA_operate.html)
