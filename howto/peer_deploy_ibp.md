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

# Deploying peers in {{site.data.keyword.cloud_notm}} Private and connect to Starter Plan or Enterprise Plan
{: #peer-ibp}


***[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

The following instructions describe how to deploy an {{site.data.keyword.blockchainfull}} Platform peer on {{site.data.keyword.cloud_notm}} Private (ICP) which will connect to Starter Plan or Enterprise Plan network on {{site.data.keyword.cloud_notm}} or your local ICP.
{:shortdesc}

Before you deploy a peer, review the [Considerations and limitations](../ibp-for-icp-about.html#ibp-icp-considerations).

Your Starter Plan or Enterprise Plan network must be running Hyperledger Fabric v1.1 or v1.2.1. You can find your Hyperledger Fabric version by opening the [Network Preferences window](../v10_dashboard.html#network-preferences) in your Network Monitor.

## Resources required
{: #peer-resources-required}

Ensure that your ICP system meets the minimum hardware resource requirements:

| Component | vCPU | RAM | Disk for data storage |
|-----------|------|-----|-----------------------|
| Peer | 2 | 2 GB | 50 GB with the ability to expand |
| CouchDB for Peer | 2| 2 GB |50 GB with the ability to expand |

 **Notes:**
 - A vCPU is a virtual core that is assigned to a virtual machine or a physical processor core if the server is not partitioned for virtual machines. You need to consider vCPU requirements when you decide the virtual processor core (VPC) for your deployment in ICP. VPC is a unit of measurement to determine the licensing cost of IBM products. For more information about scenarios to decide VPC, see [Virtual processor core (VPC) ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SS8JFY_9.2.0/com.ibm.lmt.doc/Inventory/overview/c_virtual_processor_core_licenses.html).
 - These minimum resource levels are sufficient for testing and experimentation. For an environment with a large volume of transactions, it is important to allocate a sufficiently large amount of storage; 250 GB for your peer as an example. The amount of storage to use will depend on the number of transactions and the number of signatures that are required from your network. If you are about to exhaust the storage on your peer, you must deploy a new peer with a larger file system and let it sync via your other components on the same channels.

## Storage
{: #storage}

You need to determine the storage that your peer will use. If you use the default settings, the Helm chart will create a new 8 Gi Persistent Volume Claim (PVC) with the name of `my-data-pvc` for your peer data, and another 8 Gi PVC with the name of `statedb-pvc` for your state database.

If you do not want to use the default storage settings, ensure that a *new* `storageClass` is set up during the ICP installation or the Kubernetes system administrator needs to create a storageClass before you deploy.

You can choose to deploy the peer on either the amd64 or s390x platforms. However, be aware that [Dynamic provisioning](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/) is only available for amd64 nodes in ICP. If your cluster includes a mix of s390x and amd64 worker nodes, dynamic provisioning cannot be used.

If you do not use dynamic provisioning, [Persistent Volumes ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) must be created and setup with labels that can be used to refine the Kubernetes PVC bind process.

## Prerequisites for deploying a peer
{: #prerequisites-peer-ibp}

1. Before you can install a peer on ICP, you must [install ICP](../ICP_setup.html) and [install the {{site.data.keyword.blockchainfull_notm}} Platform Helm chart](helm_install_icp.html).

2. If you use the Community Edition and you want to run this Helm chart on an ICP cluster without Internet connectivity, you need to create archives on an Internet-connected machine before you can install the archives on your ICP cluster. For more information, see [Adding featured applications to clusters without Internet connectivity ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/app_center/add_package_offline.html "Adding featured applications to clusters without Internet connectivity"){:new_window}. Note that you can find the specification file manifest.yaml under ibm-blockchain-platform-dev/ibm_cloud_pak in the Helm chart.

3. You must have an organization that is a member of a Starter Plan or Enterprise Plan network on {{site.data.keyword.cloud_notm}}. The peer leverages the API endpoints, Hyperledger Fabric CAs, and Ordering Service of the {{site.data.keyword.blockchainfull_notm}} Platform network to operate. If you are not a member of any blockchain network, you need to create or join a network. For more information, see [Creating a network](../get_start.html#creating-a-network) or [Joining a network](../get_start.html#joining-a-network).

4. You must first [deploy a CA](CA_deploy_icp.html) on ICP. You will be using this CA as a TLS CA. Follow the prerequisite steps for [operating a CA on ICP](CA_operate.html#prerequisites) before you deploy your peer. You will not need to proceed beyond those steps.

5. Retrieve the value of the cluster Proxy IP address of your TLS CA from the ICP console. **Note:** You need to be a [Cluster administrator ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/user_management/assign_role.html "Cluster administrator roles and actions") to access your proxy IP. Log in to the ICP cluster. In the left navigation panel, click **Platform** and then **Nodes** to view the nodes that are defined in the cluster. Click the node with the role `proxy` and then copy the value of the `Host IP` from the table. **Important:** Save this value and you will use it when you configure the `Proxy IP` field of the Helm chart.

6. Create a peer configuration file and store it as a Kubernetes secret in ICP. You can find the steps to create this file in the [next section](#peer-config-file).

## Building the configuration file
{: #peer-config-file}

Before you deploy a peer, you need to create configuration JSON file that contains important information about the peer identity and your certificate authority on {{site.data.keyword.cloud_notm}}. You then need pass this file to the Helm chart during configuration by using a [Kubernetes Secret ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/configuration/secret/) object. This file allows your peer to get the certificates that it needs from the certificate authority on {{site.data.keyword.cloud_notm}} to join a Starter Plan or Enterprise Plan network. This file also contains an admin certificate that allows you to operate your peer as an admin user.

We will provide you a template JSON in these instructions that you can edit and save to your local file system. We will then take you through how to use your CA to complete the configuration file.

### Configuration file

The template for the configuration file can be found below:
```
{
	"enrollment": {
		"component": {
			"cahost": "",
			"caport": "",
			"caname": "",
			"catls": {
				"cacert": ""
			},
			"enrollid": "",
			"enrollsecret": "",
			"admincerts": [""]
		},
		"tls": {
			"cahost": "",
			"caport": "",
			"caname": "",
			"catls": {
				"cacert": ""
			},
			"enrollid": "",
			"enrollsecret": "",
			"csr": {
				"hosts": [""]
			}
		}
	}
}
```
{:codeblock}

Copy this entire file into a text editor where you can edit it and save it to your local file system as a JSON file. You will only need to fill in the top two sections of this file: `"enrollment"` and `"tls"`.

To complete the configuration file, you need to compete several steps from the Network Monitor of your Starter Plan or Enterprise Plan network.

1. [Retrieve the endpoint information of your Starter Plan or Enterprise Plan CA](#ibp-ca-endpoint).
2. [Register the peer with your CA](#register-peer).
3. [Register a peer administrator](#register-admin) that you will use to operate your peer. You do not have to complete this step if you have already registered an admin to deploy another peer.

You also need to complete several steps using the Fabric CA client and your TLS CA deployed on ICP.

1. Use the Fabric CA client to [generate the peer admin MSP folder](#enroll-admin).
2. [Retrieve the endpoint information of your TLS CA](#tls-ca-endpoint).
3. Use the Fabric CA client to [register the peer with your TLS CA](#tls-register-peer).


### Starter Plan or Enterprise Plan CA information
{: #ibp-ca-endpoint}

First, we need to provide the connection information of your CA on {{site.data.keyword.cloud_notm}} to the configuration file. Log in to the Network Monitor UI on Starter or Enterprise Plan. On the **Overview** screen of your Network Monitor, click the **Remote Peer Configuration** button. This will open a pop-up containing the necessary information about your CA.

![Remote Peer Configuration](../images/myresources_starter.png "Remote Peer Configuration")
*Figure 1. Remote Peer Configuration Panel*

The pop-up window contains the following fields:

  - **Organization MSP**
  - **Certificate Authority (CA) Name**
  - **Certificate Authority (CA) URL**
  - **Certificate Authority (CA) TLS Certificate**

In the file under the `"components"` section, input the following values from above:
- `"caname"` is the value of the **Certificate Authority (CA) Name**
- `"cahost"` is the host name of the CA url. For example if the CA url is `https://ncaca9b06047b4bee966b3dec0cbb6671-org1-ca.stage.blockchain.ibm.com:31011`, the value of the `"cahost"` would be `ncaca9b06047b4bee966b3dec0cbb6671-org1-ca.stage.blockchain.ibm.com`
- `"caport"` is the port from the `"cahost"`. For example if the CA url is `https://ncaca9b06047b4bee966b3dec0cbb6671-org1-ca.stage.blockchain.ibm.com:31011`, the `"caport"` would be `31011`.
- `"cacert"`  is the value from the **Certificate Authority (CA) TLS Certificate** field. Before you insert the certificate in the file, you need to encode it to base64 format by running the following commands and replacing the string `<paste in Certificate Authority (CA) TLS Certificate>` with the value you copied from your Network Monitor.

  ```
  export FLAG=$(if [ "$(uname -s)" == "Linux" ]; then echo "-w 0"; else echo "-b 0"; fi)
  echo -e '<paste in Certificate Authority (CA) TLS Certificate>' | base64 $FLAG
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

Paste the resulting string into the `"cacert"` field  under `"catls"` in the file. After the update, the `"cacert"` field looks similar to the following example:

  ```
  "catls": {
    "cacert": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUVsRENDQTN5Z0F3SUJBZ0lRQWYyajYyN0tkY2lJUTR0eVM4KzhrVEFOQmdrcWhraUc5dzBCQVFzRkFBkOHRiUWsKQ0FVdzdDMjlDNzlGdjFDNXFmUHJtQUVTcmNpSXhwZzBYNDBLUE1icDFaV1ZiZDQ9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0KCg=="
  ```

### Register the peer
{: #register-peer}

To join your peers to channels and install and instantiate chaincode, you need to first register the peer with your CA on {{site.data.keyword.cloud_notm}}. Your peer deployment can then generate certificates that are necessary for the peer to participate in a Starter Plan or Enterprise Plan network. Complete the following steps to register a peer with an `enroll ID` and `enroll secret`. You will paste these two values into the configuration file.

1. Log in to your {{site.data.keyword.blockchainfull_notm}} Platform and access your Network Monitor. On the "Certificate Authority" screen, you can view all the identities that have been registered with your organization, such as your admin or client applications.
  ![CA screen](../images/CA_screen_starter.png "CA screen")
  *Figure 2. CA screen*

2. Click the **Add User** button on the screen. A pop-up screen opens and allows you to register your peer after you fill out the following fields. Save the values of the ID and Secret and you need to use them when you configure your peer.
  - **Enroll ID:** The user name of your peer, which is referred to as your `enroll ID` when you configure your peer. **Save this value** for the configuration file.
  - **Enroll Secret:** The password of your peer, which is referred to as your `enroll Secret` when you configure your peer. **Save this Value** for the configuration file.
  - **Type:** Select `Peer` for this field.
  - **Affiliation:** This is the affiliation under your organization, `org1` for example, that your peer will belong to. Select an existing affiliation from the drop-down list or type in a new one.
  - **Maximum Enrollments:** You can use this field to limit the number of times you can enroll or generate certificates by using this identity. If not specified, the value defaults to unlimited enrollments.

  After you complete the fields, click **Submit** to register the peer. The registered peer is then listed in the table as an identity in your organization. As a security measure, use each identity, and the accompanying enrollID and secret, to deploy only one peer. Do not reuse peer ID's and passwords.

3. In the configuration file under the `"components"` section, input the following values:
  - `"enrollid"` is the value of the `enroll ID` from the previous step.
  - `"enrollsecret"` is the value of the `enroll secret` from the previous step.


### Creating an administrator
{: #register-admin}

After you register the peer identity, you also need to create an admin identity that you will use operate to the peer. You first need to register this new identity with your CA, and use it to generate an MSP folder. You will then add the admin users signCert to the configuration file, where it will be made an admin cert of the peer during deployment. This allows you to use the certificates of the admin identity to operate your blockchain network, such as by starting a new channel or installing chaincode on your peers.

You only need to create one admin identity for the components belonging to your organization. If you are deploying multiple peers, you only need to complete these steps once. You can use the signCert that you generated for one peer to deploy any other peers belonging to your organization.

1. Log in to your {{site.data.keyword.blockchainfull_notm}} Platform and access your Network Monitor. On the "Certificate Authority" screen, you can view all the identities that have been registered with your organization, such as your admin or client applications.
  ![CA screen](../images/CA_screen_starter.png "CA screen")
  *Figure 2. CA screen*

2. Click the **Add User** button on the screen. A pop-up screen opens and allows you to register your peer after you fill out the following fields. Save the values of the ID and Secret and you need to use them when you configure your peer.
  - **Enroll ID:** The user name of your peer admin, which is referred to as your `enroll ID` when you configure your peer. **Save this value** for when you generate the admin MSP folder.
  - **Enroll Secret:** The password of your peer admin, which is referred to as your `enroll Secret` when you configure your peer. **Save this Value** for when you generate the admin MSP folder.
  - **Type:** Select `Peer` for this field.
  - **Affiliation:** This is the affiliation under your organization, `org1` for example, that your peer will belong to. Select an existing affiliation from the drop-down list or type in a new one.
  - **Maximum Enrollments:** You can use this field to limit the number of times you can enroll or generate certificates by using this identity. If not specified, the value defaults to unlimited enrollments.

  After you enter these fields, click **Submit** to create the administrator. The created administrator is then listed in the table as an identity in your organization.

### Generating the peer admin MSP folder
{: #enroll-admin}

After you register the admin identity, you need to generate the peer admin MSP folder and signCert. Therefore, you need to run an enroll command against your Starter Plan or Enterprise Plan CA.

1. Download the [Fabric CA client](CA_operate.html#fabric-ca-client) if you have not already.
2. Navigate to the directory where you will store your crypto material and create the folder where you will store the MSP folder of your peer admin.

  ```
  cd fabric-ca-client
  mkdir peer-admin
  ```
  {:codeblock}

3. Set the path where the client can create your certificates as `$FABRIC_CA_CLIENT_HOME`. Ensure that you remove the configuration material that might be created by earlier attempts. If you didn't run the `enroll` command before, the `msp` folder and the `.yaml` file do not exist.

  ```
  export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca-client/peer-admin
  rm -rf $FABRIC_CA_CLIENT_HOME/fabric-ca-client-config.yaml
  rm -rf $FABRIC_CA_CLIENT_HOME/msp
  ```
  {:codeblock}

4. Open the **Connection Profile** JSON file from your "Overview" panel in the Network Monitor, and find the following variables:
  - URL for CA: `url` under `certificateAuthorities`
  - CA Name: `caName`

5. Download the TLS certs from {{site.data.keyword.cloud_notm}} depending on the service plan, location, and cluster that you use. You can find your cluster based on the domain name of your certificate authority URL: `us01.blockchain.ibm.com:31011` or `us02.blockchain.ibm.com:31011` for example.

  - Root TLS Cert for Starter Plan
    - US: [us01.blockchain.ibm.com.cert ![External link icon](../images/external_link.svg "External link icon")](https://blockchain-certs.mybluemix.net/us01.blockchain.ibm.com.cert "us01.blockchain.ibm.com.cert"); [us02.blockchain.ibm.com.cert ![External link icon](../images/external_link.svg "External link icon")](https://blockchain-certs.mybluemix.net/us02.blockchain.ibm.com.cert "us02.blockchain.ibm.com.cert")
    - UK: [uk01.blockchain.ibm.com.cert ![External link icon](../images/external_link.svg "External link icon")](https://blockchain-certs.mybluemix.net/uk01.blockchain.ibm.com.cert "uk01.blockchain.ibm.com.cert"); [uk02.blockchain.ibm.com.cert ![External link icon](../images/external_link.svg "External link icon")](https://blockchain-certs.mybluemix.net/uk02.blockchain.ibm.com.cert "uk02.blockchain.ibm.com.cert")
    - Sydney: [aus01.blockchain.ibm.com.cert ![External link icon](../images/external_link.svg "External link icon")](https://blockchain-certs.mybluemix.net/aus01.blockchain.ibm.com.cert "aus01.blockchain.ibm.com.cert"); [aus02.blockchain.ibm.com.cert ![External link icon](../images/external_link.svg "External link icon")](https://blockchain-certs.mybluemix.net/aus02.blockchain.ibm.com.cert "aus02.blockchain.ibm.com.cert")
  - [Root TLS Cert for Enterprise Plan ![External link icon](../images/external_link.svg "External link icon")](https://blockchain-certs.mybluemix.net/3.secure.blockchain.ibm.com.rootcert)

  Save the contents to a directory where you can reference it in future commands.

    ```
    mkdir tls-ibp
    cp us01.blockchain.ibm.com.cert $HOME/fabric-ca-client/tls-ibp/tls.pem
    ```
    {:codeblock}

6. Issue the following command to generate certificates with your peer admin identity:

  ```
  fabric-ca-client enroll -u https://<enroll_id>:<enroll_password>@<ca_url_with_port> --caname <ca_name> --tls.certfiles <ca-tls_cert_file>
  ```
  {:codeblock}

  The `<enroll_id>`and `<enroll_password>` above are the **ID** and **Secret** of the peer admin that was [registered using the Network Monitor](#register-admin). The `<ca_name>` and `<ca_url_with_port>` are the values of the `caName` and `url` from your connection profile. Leave off the `http://` at the beginning of the CA URL.

  A real call might look similar to the following example command:

  ```
  fabric-ca-client enroll -u https://peeradmin:peeradminpw@n7413e3b503174a58b112d30f3af55016-org1-ca.us3.blockchain.ibm.com:31011 --caname org1CA --tls.certfiles $HOME/fabric-ca-client/tls-ibp/tls.pem
  ```
  {:codeblock}

  After this command completes successfully, it will generate a new MSP folder in the directory that you specified as `$FABRIC_CA_CLIENT_HOME`. This directory contains the certificates that you will use to operate your peer.

7. In the MSP folder that was just created, open the signCert file of the new user and convert it to the base64 format. You will find the signCert in the `signcerts` folder of the MSP directory. If you are using the example steps, you can use the following commands:

  ```
  export FLAG=$(if [ "$(uname -s)" == "Linux" ]; then echo "-w 0"; else echo "-b 0"; fi)
  cat $HOME/fabric-ca-client/peer-admin/msp/signcerts/cert.pem | base64 $FLAG
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

  This command will print out a string that is similar to the following example:

  ```
  LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNuRENDQWtPZ0F3SUJBZ0lVTXF5VDhUdnlwY3lYR2sxNXRRY3hxa1RpTG9Nd0NnWUlLb1pJemowRUF3SXcKYURFTTlEKaFhTTzRTWjJ2ZHBPL1NQZWtSRUNJQ3hjUmZVSWlkWHFYWGswUGN1OHF2aCtWSkhGeHBLUnQ3dStHZDMzalNSLwotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  ```

  Input this string to the `"admincerts"` field under the component section in the configuration file.

### TLS CA information
{: #tls-ca-endpoint}

The `"tls"` fields in the configuration file require information from the CA you deployed on ICP. You will use this CA as a separate TLS CA, which makes your peer more secure. Use the following instructions to generate the relevant information:

- The `"cahost"` and `"caport"` values are the URL and port of the [CA URL](CA_operate.html#ca-url). For example if the CA URL is `http://9.30.94.174:30167`, the value of the `cahost` would be `9.30.94.174` and the `caport` would be `30167`.
- The `"caname"` is TLS CA name of the CA that you deployed on ICP. The TLS CA name is the value that you provided to the `CA TLS instance name` field during CA configuration.
- The `"cacert"` is the base64-encoded TLS certificate of your CA. Update the following section with the value of the command output when you retrieve your [CA's TLS certificate](CA_operate.html#ca-tls) as a prerequisite.

  ```
  "catls": {
    "cacert": ""
  ```
  {:codeblock}

  After the update, the `"cacert"` field looks similar to the following example:

  ```
  "catls": {
    "cacert": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUVsRENDQTN5Z0F3SUJBZ0lRQWYyajYyN0tkY2lJUTR0eVM4KzhrVEFOQmdrcWhraUc5dzBCQVFzRkFBkOHRiUWsKQ0FVdzdDMjlDNzlGdjFDNXFmUHJtQUVTcmNpSXhwZzBYNDBLUE1icDFaV1ZiZDQ9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0KCg=="
  ```

### Registering the peer with the TLS CA
{: #tls-register-peer}

You need to register your peer with the TLS CA on ICP by using the Fabric CA client.

1. By now, you should have the TLS cert file `tls.pem` in the `$HOME/fabric-ca-client/catls` folder. If not, you can copy the TLS cert that you [downloaded from ICP](CA-operate.html#ca-tls) to a directory where you can reference it in commands below. Ensure that you're in your `$HOME/fabric-ca-client` directory.

  ```
  cd $HOME/fabric-ca-client
  mkdir catls
  cp $HOME/tls.pem $HOME/fabric-ca-client/catls/tls.pem
  ```
  {:codeblock}

2. You need to enroll by using the admin of the TLS CA. Change `$FABRIC_CA_CLIENT_HOME` to a directory where you you want to store your TLS CA admin certificates.

  ```
  cd $HOME/fabric-ca-client
  mkdir tlsca-admin
  export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca-client/tlsca-admin
  ```
  {:codeblock}

3. Run the command below to generate certificates with the TLS CA admin.

  ```
  fabric-ca-client enroll -u https://<enroll_id>:<enroll_password>@<ca_url_with_port> --caname <tls_ca_name> --tls.certfiles <ca_tls_cert_file>
  ```
  {:codeblock}

  The `<enroll_id>`and `<enroll_password>` in the command are the [CA admin user name and password](CA_deploy_icp.html#admin-secret) that you passed to the Kubernetes secret when you deployed the Certificate Authority. Insert the [CA URL](CA_operate.html#ca-url) inside the `<ca_url_with_port>`. Leave off the `http://` at the beginning. The `<tls_ca_name>` is that you specified during [CA configuration](CA_deploy_icp.html#icp-ca-configuration-parms).

  The `<ca_tls_cert_file>` is the your [CA TLS cert](CA_operate.html#ca-tls) file name with its full path.

  A real call might look similar to the following example:

  ```
  fabric-ca-client enroll -u https://admin:adminpw@9.30.94.174:30167 --caname tlsca --tls.certfiles $HOME/fabric-ca-client/catls/tls.pem
  ```
  {:codeblock}

  After you have enrolled, you have the necessary certificates to register the peer with the TLS CA.

4. Issue the following command to find your affiliation and your organization name.

  ```
  fabric-ca-client affiliation list --caname <tls_ca_name> --tls.certfiles <tlsca_tls_cert_path>
  ```
  {:codeblock}

  Your command might look like the following example:

  ```
  fabric-ca-client affiliation list --caname tlsca --tls.certfiles $HOME/fabric-ca-client/catls/tls.pem
  ```

  You should see information that is similar to the following example:

  ```
  affiliation: org1
      affiliation: org1.department1
  ```
  {:codeblock}

  Make a note of the **affiliation** value of your organization, for example, `org1.department1` for organization `org1` in the example above. You need to use this value in the command below.

5. Run the following command to register the peer.

  ```
  fabric-ca-client register --caname <tls_ca_name> --id.name <name> --id.secret <secret>  --id.affiliation org1.department1 --id.type peer --tls.certfiles <tlsca_tls_cert_path>
  ```
  {:codeblock}

  You need to use your TLS CA name in the `--caname` field.  Create a name and password for the peer and then use them to replace `name` and `secret`. **Important:** Make a note of this information. You need to input the `name` and `secret` as `"enrollid"` and `"enrollsecret"` in the `"tls"` section the configuration file.

  An example command would look similar to the following:

  ```
  fabric-ca-client register --caname tlsca --id.affiliation org1.department1 --id.name peertls --id.secret peertlspw --id.type peer --tls.certfiles $HOME/fabric-ca-client/catls/tls.pem
  ```
  {:codeblock}

  After you issue this command, you can update the file by using the values from above:

  ```
  "tls": {...
    },
    "enrollid": "peertls",
    "enrollsecret": "peertlspw",
  ```

  You can only register an identity once. If you experience a problem, try a command with a new user name and password. As a security measure, use each identity, and the accompanying enrollID and secret, to deploy only one peer. Do not reuse peer ID's and passwords.

  When the command completes successfully, you should see information that is similar to the following example:

  ```
  2018/06/18 16:53:00 [INFO] Configuration file location: /fabric-ca-client/peer-admin/fabric-ca-client-config.yaml
  2018/06/18 16:53:00 [INFO] TLS Enabled
  2018/06/18 16:53:00 [INFO] TLS Enabled
  Password: peertlspw
  ```

You can run a tree command to verify the work you have done to prepare the configuration file. Navigate to the directory where you stored your certificates. A tree command should generate a result similar to the following structure:

```
cd $HOME/fabric-ca-client
tree
.
├── ca-admin
│   ├── fabric-ca-client-config.yaml
│   └── msp
│       ├── cacerts
│       │   └── 9-12-19-115-31873-SampleOrgCA.pem
│       ├── keystore
│       │   └── c44ec1e708f84b6d0359f58ce2c9c8a289919ba81f2cf4bb5187c4ad5a43cbb0_sk
│       └── signcerts
│       |   └── cert.pem
│       └── user
├── catls
│   └── tls.pem
├── peer-admin
│   ├── fabric-ca-client-config.yaml
│   └── msp
│       ├── cacerts
│       │   └── n4790a319014a473a8a65850c660396ab-org1-ca-us05-blockchain-ibm-com-31011-org1CA.pem
│       ├── keystore
│       │   └── 6c2a999ee80a6b0592c63c58f95193bea2df59efe5489938d513de47b83b372a_sk
│       ├── signcerts
│       │   └── cert.pem
│       └── user
├── tls-ibp
│   └── tls.pem
├── tls.pem
└── tlsca-admin
    ├── fabric-ca-client-config.yaml
    └── msp
        ├── cacerts
        │   └── 9-30-250-70-30395-tlsca.pem
        ├── keystore
        │   └── bd57fa20283dfc76ada83f989ee0f62ce23e98c94dbd26f6cd23202d8084e38e_sk
        ├── signcerts
        │   └── cert.pem
        └── user
```

### CSR (Certificate Signing Request) hosts
{: #csr-hosts}

You need to provide the CSR hostnames to deploy a peer. The CSR hostnames include the proxy IP address of the cluster where you will deploy the component as well the `service host name` that will be your Helm chart host name.

#### Locating the value of the cluster proxy IP address

If you want to deploy a peer on the same ICP cluster on which you deployed your TLS CA, enter the same proxy IP that you used when you [configured for your TLS CA](CA_deploy_icp.html#icp-ca-configuration-parms). If you want to deploy the component on a different cluster, you can retrieve the value of the cluster proxy IP address from the ICP console. You need to have the cluster admin role of the ICP cluster where the peer will be deployed.

1. Log in to the ICP console. In the left navigation panel, click **Platform** and then **Nodes** to view the nodes that are defined in the cluster.
2. Click the node with the role `proxy` and then copy the value of the `Host IP` from the table.
3. Insert the `Host IP` as the value for `"hosts"` in the `"csr"` section of the configuration file below:

  ```
  "csr": {
    "hosts": [""]
  }
  ```

#### Determining the service host name

The `service host name` will be the `helm release name` that you specify during deployment. If the cluster IP Proxy address is 9.42.134.44" and the `helm release name` is `org1peer1`, you would insert the `"csr"` section of the file:

```
"csr": {
  "hosts": [
     "9.42.134.44",
     "org1peer1"
  ]
}
```
{:codeblock}

### Completing the configuration file

After you have completed all the steps, your updated configuration file will look similar to the following example:

```
{
	"enrollment": {
		"component": {
			"cahost": "n5ec276985ff54d73b9bf0e0a6525d37b-org2-ca.stage.blockchain.ibm.com",
			"caport": "31011",
			"caname": "org1CA",
			"catls": {
				"cacert": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUVsRENDQTN5Z0F3SUJBZ0lRQWYyajYyN0tkY2lJUTR0eVM4KzhrVEFOQmdrcWhraUc5dzBCQVFzRkFBkOHRiUWsKQ0FVdzdDMjlDNzlGdjFDNXFmUHJtQUVTcmNpSXhwZzBYNDBLUE1icDFaV1ZiZDQ9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0KCg=="
			},
			"enrollid": "peer",
			"enrollsecret": "peerpw",
			"admincerts": ["LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNuRENDQWtPZ0F3SUJBZ0lVTXF5VDhUdnlwY3lYR2sxNXRRY3hxa1RpTG9Nd0NnWUlLb1pJemowRUF3SXcKYURFTTlEKaFhTTzRTWjJ2ZHBPL1NQZWtSRUNJQ3hjUmZVSWlkWHFYWGswUGN1OHF2aCtWSkhGeHBLUnQ3dStHZDMzalNSLwotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg=="]
		},
		"tls": {
			"cahost": "9.30.94.174",
			"caport": "30167",
			"caname": "tlsca",
			"catls": {
				"cacert": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUVsRENDQTN5Z0F3SUJBZ0lRQWYyajYyN0tkY2lJUTR0eVM4KzhrVEFOQmdrcWhraUc5dzBCQVFzRkFBkOHRiUWsKQ0FVdzdDMjlDNzlGdjFDNXFmUHJtQUVTcmNpSXhwZzBYNDBLUE1icDFaV1ZiZDQ9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0KCg=="
			},
			"enrollid": "peertls",
			"enrollsecret": "peertlspw",
			"csr": {
        "hosts": [
           "9.42.134.44",
           "org1peer1"
        ]
			}
		}
  }
}
```
{:codeblock}

After you have completed filling in this file, you need to save it in JSON format and pass it to your peer deployment as a Kurbernetes secret. Create the secret using the steps in the [next section](#peer-config-file-ibp).

## Creating the configuration secret
{: #peer-config-file-ibp}

A [Kubernetes Secret ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/configuration/secret/) allows you to protect and share information without having to pass it directly to the deployment. Once you have saved the configuration file, you need to `base64` encode it.

1. To encode the configuration file in the base64 format, run the following commands from a terminal window:

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

  Save the resulting output for step four below.

2. Log in to the ICP console. In the left navigation panel, click **Configuration** and then **Secrets**. Click the **Create Secret** button to open a pop panel that allows you to generate a new secret object.

3. On the **General** tab, complete the following fields:
  - **Name:** Give your secret a unique name within your cluster. You will use this name when you deploy your peer. The name must be all lowercase.  
  **Note:** When you deploy a peer, a new secret is automatically generated by the deployment with the name `<helm_release_name>-secret`. Therefore, when you name your secret, be sure the name of the secret differs from the `<helm_release_name>-secret` . Otherwise, the helm chart deployment will fail because the secret it tries to create already exists.
  - **Namespace:** The namespace to add your secret. Select the `namespace` that you want to deploy your peer to.
  - **Type:** Enter the value `Opaque`.

4. Click the **Data** tab on the left navigation pane of this window. In the `Name` fields, specify `secret.json` and in the value field paste in the `base64` encoded string of your configuration file.

5. (Optional) If you plan to use `CouchDB` as your state database, you need to add two additional values to this secret object. In the **Data** tab, click the **Add data** button to add the CouchDB user ID and password to be used for the database when the container is deployed.
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
2. Click `Blockchain` in the left navigation panel to locate the tile that is labeled `ibm-blockchain-platform-prod` or `ibm-blockchain-platform-dev` if you downloaded  the Community edition. Click the tile to open it and you can see a Readme file that includes information about installing and configuring the Helm chart.
3. Click the **Configuration** tab on the top of the panel or click the **Configure** button in the lower right corner.
4. Specify the values for the [General configuration parameters](#peer-global-parameters) and accept the license agreement.
5. Open the `All parameters` twistie and specify the value for the [Global configuration parameters](#peer-global-parameters).
6. Scroll down to the **Peer configuration** section. Select the `Install Peer` check box and complete the [configuration settings](#peer-parameters) for the peer.
7. Click **Install**.

### Configuration parameters
{: #ibp-peer-configuration-parms}

The following table lists the configurable parameters of the {{site.data.keyword.blockchainfull_notm}} Platform, **specific to the peer component**, and their default values. If you are experimenting with the peer or getting started for the first time, use the default values of the database and storage parameters. Scroll to the peer component section to check the `Install Peer` checkbox and provide the associated configuration information for that component only. **Although the Helm chart UI says no further configuration is required, you need to enter certain parameters to deploy a peer.**

#### General and global configuration parameters
{: #peer-global-parameters}

|  Parameter     | Description    | Default  | Required |
| --------------|-----------------|-------|------- |
| `Helm release name`| The name of your Helm release. Must begin with a lowercase letter and end with any alphanumeric character, must only contain hyphens and lowercase alphanumeric characters. You must use a unique Helm release name each time you attempt to install a component. **Important:** This value must match the value you used to generate the 'service host name' for the "hosts" field in your [JSON secret file.](#csr-hosts) | none | yes  |
| `Target namespace`| Choose the Kubernetes namespace to install the Helm chart. | none | yes |
|**Global configuration**| Parameters which apply to all components in the Helm chart|||
| `Service account name`| Enter the name of the [service account ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/) that you will use to run the pod. | default | no |

#### Peer configuration parameters
{: #peer-parameters}

|  Parameter     | Description    | Default  | Required |
| --------------|-----------------|-------|------- |
|**Cluster configuration** |**Cluster configuration information** | ||
| `Install Peer` | Select to install a peer|unchecked | yes, if you want to install a peer |
| `Peer worker node architecture`| Select your cloud platform architecture (AMD64 or S390x)| AMD64 | yes |
| `Peer image repository`| Location of the Peer Helm chart. This field is autofilled to the installed path. If you are using the Community Edition and don't have internet access, it should match the directory where you downloaded the Fabric peer image. | ibmcom/ibp-fabric-peer | yes |
| `Peer Docker image tag`|Value of the tag associated with the peer image |1.2.1, autofilled to correct value.|yes|
| `Peer configuration`|You can customize the configuration of the peer by pasting your own `core.yaml` configuration file in this field. To see a sample `core.yaml` file, see [`core.yaml` sample config ![External link icon](../images/external_link.svg "External link icon")](https://github.com/hyperledger/fabric/blob/release-1.2/sampleconfig/core.yaml) **For advanced users only**. |none|no|
| `Peer configuration secret (Required)`| Name of the [Peer configuration secret](#peer-config-secret) you created in ICP.  |none|yes|
|`Organization MSP (Required)`|This value can be found in Network Monitor (IBP UI) by clicking "Remote Peer Configuration" on the Overview screen.  |none|yes|
|`Peer service type`| Used to specify whether [external ports should be exposed ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types) on the peer. Select NodePort to expose the ports externally (recommended), and ClusterIP to not expose the ports. LoadBalancer and ExternalName are not supported in this release. | NodePort |yes|
| `State database`| The [state database](../glossary.html#state-database) used to store your channel ledger. The peer needs to use the same database as your [blockchain network](../v10_dashboard.html#network-preferences). | LevelDB | yes |
|`CouchDB image repository`| Only applies if CouchDB was selected as the ledger database. This field is autofilled to the installed path. If you are using the Community Edition and don't have internet access, it should match the directory where you downloaded the Fabric CouchDB image.| ibmcom/ibp-fabric-couchdb | yes |
| `CouchDB Docker image tag`| Only applies if CouchDB was selected as the ledger database. Value of the tag associated with the CouchDB image. | Autofilled to correct value.| yes |
| `Peer Data persistence enabled`| Enable the ability to persist data after cluster restarts or fails. See [storage in Kubernetes ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/storage/ "Volumes") for more information.  *If unchecked, all data will be lost in the event of a failover or pod restart.* | checked | no |
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


When CouchDB is selected as the peer database, two containers are created in the pod, one for the peer and one for CouchDB. The Peer container includes a single volume mount to the Peer PVC that stores the blocks and transactions on the file system. The CouchDB container mounts the peer state database PVC that contains the ledger data.

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
{: #ibp-helm-cli}

Alternatively, you can use the Helm CLI to install the Helm release. Before you run the `helm install` command, ensure that you [add your cluster's Helm repository to the Helm CLI environment ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/app_center/add_int_helm_repo_to_cli.html "Adding the internal Helm repository to Helm CLI").

You can set the parameters required for installation by creating a `yaml` file and passing it to the following `helm install` command.

```
helm install --name <helm_release_name>  <helm_chart> \
--version <helm_chart_version> \
--values <customvalues.yaml> \
--tls
```

where:
- `<helm_release name>` represents the name that you want to give your Helm release.
- `<helm_chart>` represents the name of the Helm chart imported into the catalog.
- `<helm_chart_version>` represents the version of the Helm chart imported into the catalog.
- `<customvalues.yaml>` is the name of the yaml file that contains the configuration parameters.

For example:

```
helm install --name jnchart2 mycluster/ibm-blockchain-platform \
--version 1.1.0 \
--values peer-s390x-values.yaml \
--tls
```
You can create a new `yaml` file by editing `values.yaml` included in the downloaded archive file, which includes all the necessary parameters with their default settings.

## Verify the peer installation
{: #verify-installation-ibp}

After you complete the configuration parameters and click the **Install** button, click the **View Helm Release** button to view your deployment. If it was successful, you should see the value 1 in the `DESIRED`, `CURRENT`, `UP TO DATE`, and `AVAILABLE` fields in the Deployment table. You may need to click refresh and wait for the table to be updated. You can also find the Deployment table by clicking the **Menu** icon in the upper left corner In the ICP console. From the menu list, click **Workloads** and then **Helm Releases**.

If you scroll down to the `Notes` section, there is important information that you will use when [operating your peer](peer_operate_ibp.html).

<!--
### Verifying the peer can connect to Starter or Enterprise Plan network

You can run a peer CLI command from inside the peer container to verify that your peer has connected to your network on the {{site.data.keyword.blockchainfull_notm}} Platform. Complete the following instructions to run the `peer channel fetch` command to fetch the genesis block from a channel:

1. If you have not already connected to your ICP cluster, follow these [instructions](peer_operate_ibp.html#peer-kubectl-configure) to connect to it and use the cli from inside the peer container.

2. If you deploy your peer behind a firewall, you need to open a passthru to enable the network on the platform to complete a TlS handshake with your peer. You only need to enable a passthru for the Node port bound to port 7051 of your peer. For more information, see [finding your peer endpoint information](peer_operate_ibp.html#peer-endpoint).

3. Your organization needs to be added to a channel in the network before you can fetch the genesis block.

  - You can start a new channel for the peer. If you are using IBP Starter or Enterprise Plan, you can automatically include your organization during [channel creation](create_channel.html#creating-a-channel).

  - Another member of the blockchain network can also add your organization to an existing channel by using a [channel update](create_channel.html#updating-a-channel).

    The peer uploads its signCert during installation, so that you need to only synchronize the certificate to the channel. On the "Channels" screen of the Network Monitor, locate the channel that your organization joined and select **Sync Certificate** from the drop-down list under the **Action** header. This action synchronizes the certificates across all the peers on the channel.

4. You also need to upload the signCert of your peer admin to Starter Plan or Enterprise Plan in order for a remote CLI or application to perform channel operations such as fetching the channel genesis block and instantiating chaincode.
    1. On your local machine, cat the file `HOME/fabric-ca-client/peer-admin/msp/signcerts/cert.pem` and then copy it to the clipboard.
    2. Enter the Network Monitor of your network on {{site.data.keyword.blockchainfull_notm}} Platform.
    3. Click **Members** in the left navigator and click the **Certificates** tab.
    4. Click the **Add Certificate** button.
    5. In the **Add certificate** window, give your certificate a name, such as "peer-admin", paste in the certificate you just copied to the clipboard and click **Submit**.
    6. You will be asked whether you want to restart the peers. Click **Restart**.
    7. Click **Channels** in the left navigator and locate the channel that the peer will join.
    8. Click the three dots under the **Actions** header and click **Sync Certificate**. In the **Sync certificate** window, click **Submit**.

5. Retrieve configuration information from your Connection profile available on the "Overview" screen of your Network Monitor. Click **Connection Profile** and then **Download**.

  - Find the orderer URL by searching for **orderers**, which is located under `orderers > url`. Make a note of the URL that begins with the network name. The URL is similar to the following example:

    ```
    ash-zbc07b.4.secure.blockchain.ibm.com:21239
    ```

  - Find the name of your organization by searching for **organizations**. This should be the same organization as you use to register your peer. You can find your organization's name together with its associated `mspid`. Make a note of the value of the `mspid`.

6. Run the following commands to set the environment variables in the peer container.

  ```
  export ORDERER_1=<ORDERER_URL>
  export CHANNEL=<CHANNEL_NAME>
  export CC_NAME=<CC_NAME>
  export ORGID=<ORGANIZATION_MSP_ID>
  export PEERADDR=<PEER_ADDR>
  ```
  {:codeblock}

  Replace the fields with your own information.
  - Replace `<ORDERER_URL>` with the hostname and port of the orderer from the `creds.json` file.
  - Replace `<CHANNEL_NAME>` with the name of the channel that the peer joins.
  - Replace `<CC_NAME>` with any name to refer to your chaincode.
  - Replace `<ORGANIZATION_MSP_ID>` with the name of the organization from the `creds.json` file.
  - Replace `<PEER_ADDR>` with `localhost:7051`

  For example:

  ```
  export ORDERER_1=ash-zbc07b.4.secure.blockchain.ibm.com:21239
  export CHANNEL=defaultchannel
  export CC_NAME=mycc
  export ORGID=org1
  export PEERADDR=localhost:7051
  ```
  {:codeblock}

7. Run the following peer CLI command to fetch the genesis block of the channel.

  ```
  CORE_PEER_TLS_ROOTCERT_FILE=/mnt/certs/tls/cacert.pem CORE_PEER_TLS_ENABLED=true CORE_PEER_ADDRESS=${PEERADDR} CORE_PEER_LOCALMSPID=${ORGID} CORE_PEER_MSPCONFIGPATH=/mnt/crypto/peer/peer/msp/ GOPATH=/ peer channel fetch 0 -o ${ORDERER_1} -c ${CHANNEL} --cafile /mnt/certs/tls/cacert.pem --tls
  ```
  {:codeblock}

  **Note:** It is possible that when you run any of these CLI commands, you might experience the following warning that can be safely ignored.

  ```
  [msp] getPemMaterialFromDir -> WARN 001 Failed reading file
  /mnt/crypto/peer/peer/msp/intermediatecerts/<intermediate cert name>.pem: no pem content for file /mnt/crypto/peer/peer/msp/intermediatecerts/<intermediate cert name>.pem
  ```

  Verify that the command worked successfully and that genesis block is added to your peer container by running the following command:

  ```
  ls *.block
  ```
  {:codeblock}

  You know that the genesis block is added successfully when you see something that is similar to the following example:

  ```
  defaultchannel_0.block
  ```

  Successfully fetching the genesis block indicates that your peer can connect to your Starter or Enterprise Plan network.
  You still need to join the peer to the channel and install chaincode. See [operating your peer](peer_operate_ibp.html) for more information.

-->

## Viewing the peer logs
{: #peer-ibp-view-logs}

Component logs can be viewed from the command line by using the [`kubectl CLI commands`](peer_operate_ibp.html#peer-kubectl-configure) or through [Kibana ![External link icon](../images/external_link.svg "External link icon")](https://www.elastic.co/products/kibana "Your window into the Elastic Search"), which is included in your ICP cluster. For more information, see these [instructions for accessing the logs](peer_operate_ibp.html#peer-ibp-view-logs).

## What's next

After you have deployed the peer, you need to complete several operational steps before you can submit transactions to and read the distributed ledger from the blockchain network. For more information, see [Operating peers with Starter Plan or Enterprise Plan](peer_operate_ibp.html).
