---

copyright:
  years: 2019, 2021
lastupdated: "2021-05-11"

keywords: network components, IBM Cloud Kubernetes Service, allocate resources, batch timeout, reallocate resources, LevelDB, CouchDB

subcollection: blockchain

---

{:external: target="_blank" .external}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:note: .note}
{:important: .important}
{:tip: .tip}
{:pre: .pre}


# Upgrading and deleting deployed nodes
{: #ibp-console-govern-components}



After creating CAs, peers, and ordering nodes, you need to monitor the resources used by the nodes and potentially reallocate resources.
{:shortdesc}

**Target audience:** This topic is designed for network operators who are responsible for creating, monitoring, managing, and upgrading their components in the blockchain network.

After a node has been created, there are three main ways to update it.

1. **Update its configuration**. Just as it is possible to override configuration parameters when deploying a node, it is possible to edit many parameters and redeploy the node. For more information, see our topic on [Advanced deployment options](/docs/blockchain?topic=blockchain-ibp-console-adv-deployment).
2. **Increase the resources allocated to it**. If you notice that the performance of a node is beginning to lag or that its storage is beginning to run out, you can choose whether to deploy a larger node and join it to channels or whether to increase the resources allocated to your existing node.
3. **Upgrade its Fabric version**. Every node is deployed using a release version of Hyperledger Fabric. When new versions of Fabric become available on the IBM Blockchain Platform, **Upgrade available** is visible on the tile representing the node. While upgrading to a new version of Fabric is always recommended, unless is rendered necessary by the capabilities of channels you want to join, it is typically optional. For more information, check out [Upgrading to a new version of Fabric](#ibp-console-govern-components-upgrade).

In this topic, we'll cover the process for [reallocating resources](#ibp-console-govern-components-reallocate-resources), upgrading to a new version of Fabric, and [deleting nodes](#ibp-console-govern-components-delete).

## Considerations when reallocating resources
{: #ibp-console-govern-components-reallocate-resources}

Resizing a node requires the containers to be rebuilt, which can cause a delay in the functioning of the node.
{:important}

We recommend using the [{{site.data.keyword.mon_full_notm}}](https://www.ibm.com/cloud/cloud-monitoring) tool in combination with your {{site.data.keyword.cloud_notm}} Kubernetes dashboard to monitor your Kubernetes resource usage. When you determine that a worker node is running out of resources, you can add a new larger worker node to your cluster and then delete the existing working node.
{:note}




While it takes less effort to deploy enough resources to your Kubernetes cluster from the start and therefore be able deploy and expand resources without having to increase the resources in your cluster, the bigger the deployment, the more it will cost. Users need to consider their options carefully and recognize the tradeoffs that they are making regardless of the option that they choose.


You can scale your cluster manually by monitoring your nodes and either adding more nodes or larger nodes. While this process can be labor intensive, it has the advantage of allowing the user to always be certain what is being charged to their cloud account.

A Kubernetes cluster on {{site.data.keyword.cloud_notm}} includes the ability to use an **autoscaler** that can scale your worker nodes up or down in response to your pod spec settings and resource requests. For more information about the autoscaler and how to set it up, see [Scaling Kubernetes clusters](/docs/containers?topic=containers-ca#ca){: external} or [Scaling OpenShift clusters](/docs/openshift?topic=openshift-ca){: external} in the {{site.data.keyword.cloud_notm}} documentation. Note that allowing the autoscaler to adjust your resources will result in charges to your {{site.data.keyword.cloud_notm}} account that will vary automatically with your usage.

To scale manually in the console, click the node that you want to adjust on the **Nodes** page and then click the **Usage** tab. You can see a button called **Reallocate**, which will launch a **Resource allocation** tab that is very similar to the one that you saw when you created the node. If you want to lower the amount of available resources, simply provide lower values and click **Reallocate resources** on that tab and the resulting **Summary** page.

If you want to increase the CPU and memory for a node, use the **Resource allocation** panel in the console to increase the values. The white box at the end of the page adds up the new values. After clicking **Reallocate resources**, the **Summary** page will translate this value into a **VPC** amount, which is used to calculate your bill. You'll then need to navigate to your Kubernetes cluster to make sure your cluster has sufficient resources for this reallocation. If it does, you can click **Reallocate resources**. If sufficient resources are not available, you must increase the size of your cluster.

The method you will use to increase storage will depend on the storage class you chose for your cluster.  Refer to the [Kubernetes](/docs/containers?topic=containers-kube_concepts#kube_concepts){: external} or [OpenShift](/docs/openshift?topic=openshift-kube_concepts) storage options in the {{site.data.keyword.cloud_notm}} documentation. If you are about to exhaust the storage on your peer or ordering node, you might need to deploy a new peer or ordering node with a larger file system and let it sync via your other components on the same channels.

In {{site.data.keyword.cloud_notm}}, CPU and memory can be increased using the console if you have resources available in your Kubernetes cluster on {{site.data.keyword.cloud_notm}}. However, storage must be increased using the {{site.data.keyword.cloud_notm}} CLI. For a tutorial on how to do this, see Changing the size and IOPS of your existing storage device on your [Kubernetes](/docs/containers?topic=containers-file_storage#file_change_storage_configuration){: external} or [OpenShift](/docs/openshift?topic=openshift-block_storage#block_change_storage_configuration){: external} cluster.




Note that you do not need to adjust the CPU, memory, or storage for your smart contract pods. These pods will automatically use as many resources as they need to function efficiently. In cases where your smart contracts are struggling because of insufficient resources, you will have to address this at the cluster level.
{: tip}


### Monitoring file storage
{: #ibp-console-govern-components-monitor-storage}

If you are using **Classic infrastructure**, to view your consumption of file storage, navigate to your Kubernetes cluster on {{site.data.keyword.cloud_notm}}. Click on the menu icon ![action menu](../../icons/overflow-menu.svg). Then click **Classic infrastructure** > **Storage** > **File Storage**. This will display the capacity and usage for each persistent volume claim (PVC). This usage can be mapped to your {{site.data.keyword.blockchainfull_notm}} Platform nodes by clicking on the cell in the **Notes** column.

You will see something that looks like this:

```
PVC {"plugin":"ibm-file-plugin-77497497cc-5qm58","region":"us-south","cluster":"05ccfca248dc4c389978d074524e0a8e","type":"Endurance","ns":"n4c817f","pvc":"n4c817forg1ca-fabric-ca-pvc","pv":"pvc-63074ac8-8b9b-11e9-88db-222bd59fb4bc","storageclass":"default","reclaim":"Delete"}
```

To see the same information from the command line, log in to your cluster and enter this command:

```
ibmcloud sl file volume-list --column id --column notes
```

This will allow you to map the output from the pods to the nodes you have deployed.

If you are using VPC infrastructure, see [About block storage for VPC](/docs/vpc?topic=vpc-block-storage-about) for more details.

### Adding storage
{: #ibp-console-govern-components-add-storage}

As you monitor your pods and notice that more storage is needed, you can increase storage when additional storage is available. The following links provide more information about how to increase storage after your network is deployed.

- [IBM file storage](/docs/FileStorage?topic=FileStorage-expandCapacity)
- [Portworx](https://docs.portworx.com/portworx-install-with-kubernetes/storage-operations/create-pvcs/resize-pvc/)
- [Block storage](/docs/BlockStorage?topic=BlockStorage-expandingcapacity#expandingcapacity)

Users can allocate more storage to their running network by resizing the existing storage PVCs or by deploying nodes with new PVCs.


## Upgrading to a new version of Fabric
{: #ibp-console-govern-components-upgrade}

When updating {{site.data.keyword.blockchainfull_notm}} Platform nodes from 1.4.x to 2.x.x, select 2.2.x or the latest version available. Do not select any version prior to 2.2.x. See the [Fabric documentation on upgrading](https://hyperledger-fabric.readthedocs.io/en/release-2.2/upgrade_to_newest_version.html#upgrading-to-2-2-from-the-1-4-x-long-term-support-release){: external} for more information.
{: important}

While some new versions of Fabric are released where only the Fabric version of nodes must be upgraded in order to get the latest Fabric features, some new Fabric versions contain new channel capabilities that must also be updated.

In these cases, the process of "updating to the latest" release is, at a high level, a two step process:

1. Upgrade the Fabric version of all nodes.
2. Update the channels to the new capability levels. For information about how to edit channels to use the latest capabilities, check out [Capabilities](/docs/blockchain?topic=blockchain-ibp-console-govern#ibp-console-govern-capabilities).

You must upgrade nodes before you update the channels. If a node attempts to read a configuration block containing a capability level it does not understand (which is true in cases where the capability is a higher level than the node version), the node will crash **on all channels**. The node must then be upgraded to the appropriate Fabric version before it can be used again.
{: important}

At a high level, the process of upgrading a node involves two main steps:

1. Backing up the persistent volumes associated with the node. These backups ensure that in the case of an upgrade failure in which the peer pod is corrupted that the node can be re-deployed using the ledger. For more information, see [Backup considerations for each node type](/docs/blockchain?topic=blockchain-backup-restore#backup-restore-node-considerations).
2. Upgrading the Fabric versions of the nodes one at a time (also known as a "rolling upgrade").

It may also be necessary to update SDKs and smart contracts before you can take advantage of the latest Fabric features. For more information, check out [Step three: Update SDKs and smart contracts](#ibp-console-govern-components-upgrade-step-three).
{: tip}

### Step one: Backup your ledger
{: #ibp-console-govern-components-upgrade-step-one-ledger}

It is recommended to take regular backups of the persistent volumes of your nodes as part of the normal process of network administration. For example, in our topic on backing up and restoring components and networks, an [example schedule](/docs/blockchain?topic=blockchain-backup-restore#backup-restore-schedule-snapshot) is provided which recommends that backups be taken of the ordering nodes and the peers (including the state database of the peer, if using CouchDB) each night.

However, if you are not taking regular backups, it is recommended that you minimally take a backup before attempting to upgrade a node, as it allows for a node to be restored to an earlier running state in cases where the upgrade fails. For information about how to backup your nodes by taking a snapshot of the relevant persistent volumes, check out [Taking snapshots](/docs/blockchain?topic=blockchain-backup-restore#backup-restore-take-snapshot).

### Step two: Upgrade your nodes one at a time
{: #ibp-console-govern-components-upgrade-step-two-rolling-upgrade}

If you are upgrading both peer and ordering node binaries, it is a best practice to upgrade the ordering nodes first, as ensuring that the ordering nodes (and by extension, the ordering service) is functioning correctly is more important to the health of your network as a whole than the functioning of any particular peer.
{: tip}

The process for upgrading a node is relatively straightforward. First, make sure you are using the console where the node was created. You cannot use the console to update imported nodes. When a node upgrade is available, **Upgrade available** is visible on the node tile. If **Upgrade available** does not appear on the tile when it should be there, make sure your console is using the latest version using the [**Refresh cluster**](/docs/blockchain?topic=blockchain-ibp-console-manage-console#ibp-console-refresh) feature. <blockchain-sw-25>If **Upgrade available** does not appear on the tile when it should be there, make sure you have upgraded to the latest version of the console.</blockchain-sw-25>

You can then update the node:

1. Click on the tile representing the node.
2. The upgrade panel can be accessed either by clicking the **Info and usage** tab, next to the **Upgrade available** notice, or the **Fabric version** box. Clicking either one opens the **Upgrade Fabric version** side panel.
3. On the **Upgrade Fabric version** side panel, review the version of your node and the version you are upgrading to. If this is right, click **Next**.
4. On the next panel, confirm the information and enter the name of the node being upgraded.

The node will be unavailable during the upgrade. The status was turn grey and will read **Status unknown** or **Unavailable**. When the upgrade has completed, the status will turn green and be **Ready**. If the upgrade fails and the node lapses into an unrecoverable state, follow the instructions for [Restoring nodes](/docs/blockchain?topic=blockchain-backup-restore#backup-restore-restore) from a snapshot.

To upgrade from a v1.4.x peer to a 2.x peer, the databases of all peers (which include not just the state database but the history database and other internal databases for the peer) must be rebuilt using the v2.x data format. Depending on the size of your ledgers and the number of channels your peer is joined to, the process of rebuilding your state databases can take quite a bit of time (several hours, in some cases), and is a mandatory part of the upgrade process. The time it takes to upgrade varies not just on the number of blocks, but on your network speed and the average size of the blocks in your channels. Note that the node is down during upgrade as the pods are rebuilt. When the upgrade has completed, access to the node is restored.
{: tip}

**It is a best practice to only upgrade one node of each type at a time**. In other words, if you need to upgrade both peers and ordering node, it is alright if you start a peer upgrade and an ordering node upgrade at the same time. However, do not attempt to upgrade multiple peers or multiple ordering nodes at the same time, as this threatens the availability of your components.

It is not possible to upgrade an ordering node if any node in your ordering service is down for any reason. Coordinate with all of the administrators of your ordering service before attempting to upgrade.

### Step three: Update SDKs and smart contracts
{: #ibp-console-govern-components-upgrade-step-three}

It is a best practice to upgrade your SDK to the latest version as part of a general upgrade of your network. While the SDK will always be compatible with equivalent releases of Fabric and lower, it might be necessary to upgrade to the latest SDK to leverage the latest Fabric features. Also, after upgrading, it's possible your client application may experience errors. Consult the your Fabric SDK documentation for information about how to upgrade.
{: tip}

Although support for Fabric 2.0 networks was added to the platform, you can still run your existing smart contracts on your peers that run a v1.4 image on a channel with an application capability level of 1.4 or lower. Should you later decide to upgrade your peer to a v2.x image and update your channel application capability level to 2.0, **you may need to update your existing smart contract**. However, after you upgrade your peer image to v2.x and channel application capability v2.x, there is no longer a way to update the original smart contract. Instead, when an update is required, you need to repackage the smart contract in the new `.tar.gz` or `.tgz` format using v2 of the VS Code extension and then propose the definition to the channel using the new smart contract lifecycle process.  

Review the following considerations:  

**Node**  

If your smart contract was written in Node, then you might need to update it. By default, a Fabric v1.4 peer will create a Node v8 runtime, and a Fabric v2.x peer creates a Node v12 runtime. In order for a smart contract to work with Node v12 runtime, the `fabric-contract-api` and `fabric-shim` node modules must be at v1.4.5 or greater. If you are using a smart contract that was originally written to work with Fabric 1.4, update the Node modules by running the following command before deploying the smart contract on a Fabric v2.x peer.  See [Support and compatibility for fabric-chaincode-node](https://github.com/hyperledger/fabric-chaincode-node/blob/main/COMPATIBILITY.md) for more information.
```
npm install --save fabric-contract-api@latest-1.4 fabric-shim@latest-1.4
```
{: codeblock}

**Go**  

Because Fabric v2.x peers do not have a "shim" (the external dependencies that allowed smart contracts to run on earlier versions of Fabric), you need to vendor the shim and then repackage any smart contracts written in Golang (Go) that use the [Go SDK](https://github.com/hyperledger/fabric-sdk-go). "Vendoring the shim"  effectively means that you are copying the dependencies into your project. Without this vendoring and repackaging, the Go smart contract cannot run on a peer using a Fabric 2.x image. If you are using the [IBM Developer Tools](/docs/blockchain?topic=blockchain-develop-vscode) to develop and package your smart contract, the tooling performs the vendoring for you. This process is not required for smart contracts that are written in Java or Node.js, nor for Go smart contracts that use the [Go contract-api](https://github.com/hyperledger/fabric-contract-api-go){: external}.

**Java**  
The `build.gradle` file for the smart contract must be updated:

1. If the smart contract uses the `shadowjar` 2.x plugin, then it should be updated to version 5 by using the following code:
	```
	plugins {
	    id 'com.github.johnrengelman.shadow' version '5.1.0'
	    id 'java'
	}
	```
	{: codeblock}

2. The `repositories` section of the file must contain the `maven URL` for `jitpack`, for example:
  ```
	repositories {
	    ...
	    maven {
	        url 'https://jitpack.io'
	    }
	}
	```
	{: codeblock}


**Init functions**  

If the smart contract was written using the **low-level APIs** provided by the Fabric Chaincode Shim API, your smart contract needs to contain an `Init` function that is used to initialize the chaincode.  This function is required by the smart contract interface, but does not necessarily need to be invoked by your applications. Because you cannot use the console to deploy a smart contract that contains an `Init` function, you need to move that initialization logic into the smart contract itself and call it separately. For example, the smart contract can use a reserved key to check whether the smart contract has already been initialized or not. If not, then call the initialization logic, otherwise proceed as usual. If your smart contract needs to include the `Init` function, the only way to deploy it is by using the Fabric [peer lifecycle chaincode install](https://hyperledger-fabric.readthedocs.io/en/release-2.2/commands/peerlifecycle.html#peer-lifecycle-chaincode-install){: external} command or the [{{site.data.keyword.blockchainfull_notm}} Platform collection for Ansible](https://ibm-blockchain.github.io/ansible-collection/){: external}. You can also refer to the [Fabric documentation](https://hyperledger-fabric.readthedocs.io/en/release-2.2/chaincode_lifecycle.html#step-three-approve-a-chaincode-definition-for-your-organization){: external} for more details on how to use an `Init` function with the Fabric chaincode lifecycle.

**Repackage smart contract**  

After you have updated your smart contract, use [v2](/docs/blockchain?topic=blockchain-develop-vscode#develop-vscode-installing-the-extension) of the VS Code extension to [repackage](/docs/blockchain?topic=blockchain-develop-vscode#packaging-a-smart-contract) your smart contract.     

For a look at how the new lifecycle is administered in the console, check out [Deploy a smart contract using Fabric v2.x](/docs/blockchain?topic=blockchain-ibp-console-smart-contracts-v2). For a look at the possibilities the new lifecycle opens up, check out [Writing powerful smart contracts](/docs/blockchain?topic=blockchain-write-powerful-smart-contracts).

### Step four: Update capabilities
{: #ibp-console-govern-components-upgrade-step-four-rolling-upgrade}

Once your nodes, SDKs, and smart contracts have been upgraded to use the latest Fabric version, you can update your channel configuration to use the latest capabilities. Note that the Fabric version of your nodes must be at least at the corresponding capability level of the channel the node is joined to.

For more information about capabilities and how to update a channel configuration to enable them, check out [Capabilities](/docs/blockchain?topic=blockchain-ibp-console-govern#ibp-console-govern-capabilities).

## Deleting components
{: #ibp-console-govern-components-delete}

The best practice for deleting components is to delete them using the console. This will also delete all of the artifacts associated with a node including your ledger data in persistent storage and the keys that are stored as secrets. Deleting a peer will not, however, delete any smart contract pods associated with it. These must be deleted separately. Deleting a component is usually achieved by logging onto the console where a component was created or installed, clicking on the component and finding the related **trash can** icon. You will typically be prompted to type the name of the component and to confirm your decision. You can also delete nodes by using the {{site.data.keyword.blockchainfull_notm}} Platform APIs.

However, there are cases in which this type of deletion will not be successful. For example, occasionally when a node fails to deploy it will not be possible to delete it using the console. The same can be true if the console loses connection with the cluster for some reason.

In these cases, it will be necessary to use the Kubernetes dashboard or delete the node or relevant pods manually. If you attempt to delete pods that contain deployments such as peers, CAs, or ordering nodes, the pod will automatically restart and not be permanently deleted. 

Because smart contracts installed on a 2.x peer are deployed into their own pods and not directly into the peer container, they will not be deleted when a peer is deleted. They will have to be deleted either using the UI of your cluster or by issuing kubectl commands. Smart contracts installed on a v1.4.x peer will be deleted when the peer is deleted.
{: important}




Because deployments are organized in Kubernetes by their "namespace", you need to know your Kubernetes cluster namespace before you can delete components using the kubectl CLI. From the console, open any CA node and click the **Info and Usage** icon. View the value of the API URL. For example: https://nf85a2a-soorg10524.ibpv2-cluster.us-south.containers.appdomain.cloud:7054. The namespace is the first part of the URL beginning with the letter `n` and followed by a random string of six alphanumeric characters. So in the example above the value of the namespace is `nf85a2a`.


If you want to delete all of your smart contract pods, you can issue this command:


```
kubectl get po -n <NAMESPACE> | grep chaincode-execution | cut -d" " -f1 | xargs -I {} kubectl delete po {} -n <NAMESPACE>
```
{:codeblock}




If you want to delete a single smart contract pod, you will first have to figure out the name of your smart contract pod.


First, get a list of all of the smart contract pods running in your cluster:

```
kubectl get po -n <NAMESPACE> | grep chaincode-execution | cut -d" " -f1 | xargs -I {} kubectl get po {} -n <NAMESPACE> --show-labels
```
{:codeblock}

Replacing `<NAMESPACE>` with the name of your cluster namespace.

You should see results similar to:

```
NAME                                                       READY   STATUS    RESTARTS   AGE   LABELS
chaincode-execution-0a8fb504-78e2-4d50-a614-e95fb7e7c8f4   1/1     Running   0          14s   chaincode-id=javacc-1.1,peer-id=org1peer1
NAME                                                       READY   STATUS    RESTARTS   AGE   LABELS
chaincode-execution-f3cc736f-94ef-454d-8da3-362a50c653d9   1/1     Running   0          4m    chaincode-id=nodecc-1.1,peer-id=org1peer1
```

Your smart contract name and version is visible next to the chaincode-id.





To delete a single pod, issue this command, substituting the `<POD_NAME>` for the name of your pod, for example the smart contract pod `chaincode-execution-0a8fb504-78e2-4d50-a614-e95fb7e7c8f4`, as well as your `<NAMESPACE>`:

```
kubectl delete pod <POD_NAME> -n <NAMESPACE>
```
{:codeblock}





You can also use kubectl commands to delete all of the nodes in your cluster by issuing commands to delete each type of node. First, set the namespace where the nodes you want to delete are located:

```
kubectl config set-context --current --namespace=<NAMESPACE>
```
{:codeblock}




Then run the following commands to delete all of your blockchain nodes:

```
kubectl delete ibpca --all
kubectl delete ibppeer --all
kubectl delete ibporderer --all
```
{:codeblock}

You may also choose to only delete all of a single type of node within a namespace, for example, by only issuing `kubectl delete ibppeer --all`.

Note that if you delete your entire namespace, your smart contract pods will also be deleted. Your smart contract pods will also be deleted, along with your nodes, if you delete your service instance using the Kubernetes dashboard in {{site.data.keyword.cloud_notm}}.
{: tip}
