---

copyright:
  years: 2018
lastupdated: "2018-12-07"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}

# Operating a Certificate Authority on {{site.data.keyword.cloud_notm}} Private
{: #ca-operate}

***[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***

Certificate authorities (CAs) provide identities on the network. A CA can be considered as being similar to a publicly trusted notary that is used to establish trust among multiple parties. Each entity in the network is given a certificate that a root CA signs to encapsulate the entity's digital identity. This certificate is the root of trust for all the signing and verification operations that are performed on the network.
{:shortdesc}

Each organization in a multi-cloud blockchain network should deploy its own CA. Your organization's CA will sign requests for the components that your organization operates, such as an ordering service, peers, or applications. Therefore, you need to deploy your CA before you deploy any other components.

You need to take a few prerequisite steps to operate your Certificate Authority:

- [Configure your CLIs](#ca-kubectl-configure)
- [Retrieve your Certificate Authority URL](#ca-url)
- [Download your CA TLS cert](#ca-tls)
- [Set up the Fabric CA client](#fabric-ca-client)
- [Generate certificates with your CA admin](#enroll-admin)

After you complete the prerequisite steps, you can use your CA to register new identities in the network with your organization and to generate certificates that your applications can use.

- [Using the CA to deploy an orderer or peer](#deploy-orderer-peer)
<!-- [Register and enroll client applications](#register-enroll-app)-->

This guide also provides some important conceptual information about operating your CA and how to use its certificates to manage a blockchain network.

- [Membership Service Providers (MSPs)](#msp)
- [TLS certificates](#tls)

## Prerequisites
{: #prerequisites}

Whether you are founding or joining a network, you need to complete these prerequisites steps to use your CA and deploy other components and identities that belong to your organization.

### Configuring the CLIs
{: #ca-kubectl-configure}

You need to use the **kubectl** command line tool to connect to CA container that runs in ICP.

1. Log in to your ICP cluster UI. Navigate to the **Command Line Tools** tab and click **Cloud Private CLI**. You'll see the following tools that you can download.

   * Install IBM Cloud Private CLI and plug-ins
   * Install Kubectl CLI
   * Install Helm
   * Install Istio CLI

  To operate a CA, you need to use the first **three** tools, among which Helm is optional. Click each of them and run the `curl` commands for the machine type that you're using. Then, issue `chmod` and `sudo mv` commands for each tool. The `chmod` command will change the permission of the CLI in question to make it executable, and the `sudo mv` command will move the file and rename it.

  The commands for the first tool **cloudctl** might look like the following example:

  ```
  chmod +x cloudctl-darwin-amd64<suffix of your binary>
  sudo mv path/to/cloudctl-darwin-amd64<suffix> /usr/local/bin/cloudctl
  ```
  {:codeblock}

  The commands for the second tool **kubectl** might look like the following example:

  ```
  chmod +x kubectl-darwin-amd64<suffix of your binary>
  sudo mv path/to/kubectl-darwin-amd64<suffix> /usr/local/bin/kubectl
  ```
  {:codeblock}

2. After you configure **kubectl**, run the following command:

  ```
  kubectl get pods
  ```
  {:codeblock}

  If successful, the command should return a response that is similar to the following example, where `ca-6cf85f6687-hcxpl` is the name of the CA container.

  ```
  NAME                                 READY      STATUS             RESTARTS   AGE
  ca-6cf85f6687-hcxpl          2/2        Running            0          6h
  ```

  You are now ready to use the **kubectl** tool to get the CA URL.

3. Optionally, if you want to use **Helm**, complete a few more steps. Note that Helm is optional to install and you don't need to use it in these instructions. However, it can be useful to manage your Helm releases and create your own archives to deploy in ICP.

  1. Click "Install Helm" and run the `curl` command from the ICP UI.
  2. Unpack the `tar` file by running the following command:  
    ```
    tar -xzvf helm-darwin-amd64<suffix>
    ```
    {:codeblock}
  3. Run the `sudo mv` command by appending `/helm` to `darwin-amd64`. Note that you do not need to run the `chmod` command for Helm. For example:  
    ```
    sudo mv darwin-amd64/helm/ /usr/local/bin/helm
    ```
    {:codeblock}

  You can run the `helm help` command to confirm that Helm is installed successfully.

### Retrieving your Certificate Authority URL
{: #ca-url}

You need to target the CA URL for requests to generate certificates or register with a new identity. You can find your CA URL by using your ICP console UI. You will need to be a [Cluster administrator ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/user_management/assign_role.html "Cluster administrator roles and actions") to complete the following steps:

1. Log in to your ICP console and click the **Menu** icon in the upper left corner.
2. Click **Workload** > **Helm Releases**.
3. Find the name of your Helm Release and open the Helm Release details panel.
4. Scroll down to the **Notes** section at the bottom of the panel. In the **Notes** section, you can see a set of commands to help you operate your CA deployment.
5. If you have not already, follow the steps to configure the [kubectl CLI](#ca-kubectl-configure), which you need to use to interact with your CA container.
6. In your CLI, run the first command in the note, which follows **1. Get the application URL by running these commands:** This command prints out the CA URL and port, which is similar to following example. Save this value as the application URL to use with subsequent commands.

  ```
  http://9.30.94.174:30167
  ```

**Note:** If you deploy your CA behind a firewall, you need to open a passthru to enable the network on the platform to complete a TLS handshake with your CA. You need to enable a passthru for only the port that was concatenated with the CA URL.


### Retrieving your CA TLS cert
{: #ca-tls}

You need to download your CA TLS certificate and pass it with along with your commands to communicate with your CA from a remote client.

1. Log in to your ICP console. Click the **Menu** icon in the upper left corner.
2. Click **Workload** > **Helm Releases**.
3. Find the name of your Helm Release and open the Helm Release details panel.
4. Scroll down to the **Notes** section at the bottom of the panel. In the **Notes** section, you can see a set of commands to help you operate your CA deployment.
5. If you have not already, follow the steps to configure the [kubectl CLI](#ca-kubectl-configure), which you need to use to interact with your CA container.
6. In your CLI, run the commands in the third note, which follows **3. Get TLS certificate:** This command saves your TLS certificate as the file `tls.pem` on your local directory. You need to reference this certificate in a future command. Make note of its location.
7. The command also converts the certificate into base64 format and print it out. The result looks similar to the string below:

  ```
    LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUVsRENDQTN5Z0F3SUJBZ0lRQWYyajYyN0tkY2lJUTR0eVM4KzhrVEFOQmdrcWhraUc5dzBCQVFzRkFBkOHRiUWsKQ0FVdzdDMjlDNzlGdjFDNXFmUHJtQUVTcmNpSXhwZzBYNDBLUE1icDFaV1ZiZDQ9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0KCg==
  ```

  **Important:** You need to copy this string and save it as your `ICP CA TLS certificate`. You will use it when you deploy additional components.

### Setting up the Fabric CA client
{: #fabric-ca-client}

You can use the Fabric CA client to operate your CA. These instructions explain how to use the Fabric CA client to enroll and register identities for other components that belong to your organization. You can also use the Fabric SDKs to complete the prerequisite steps.

1. You need to download the [Fabric CA client ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#fabric-ca-client "Download Fabric CA client") to your local file system.

  The easiest way to get the Fabric CA client is to download all of the Fabric tool binaries directly. Navigate to a directory where you would like to download the binaries with your command line, and fetch them by running the following [Curl ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/release-1.2/prereqs.html#install-curl "Curl") command.

  ```
  curl -sSL http://bit.ly/2ysbOFE | bash -s 1.2.1 1.2.1 -d -s
  ```
  {:codeblock}

  This command installs the binaries in a `bin/` directory.

2. Set the `PATH` path to directory where you downloaded the Fabric tool binaries:

  ```
  export PATH=$PATH:<full/path/to/fabric-client/bin>
  ```
  {:codeblock}

  For example, if you installed the binaries in your home directory you would set your `PATH` as:

  ```
  export PATH=$PATH:$HOME/bin
  ```
  {:codeblock}

3. Create the folder where you will store the certificates of your CA admin.

  ```
  mkdir -p $HOME/fabric-ca-client/ca-admin
  ```
  {:codeblock}

4. Set the value of the `$FABRIC_CA_CLIENT_HOME` environment variable to be the path where the CA client will store the generated [MSP](#msp) certificates. Ensure that you remove the configuration material that might be created by earlier attempts. If you didn't run the `enroll` command before, the `msp` folder and the `.yaml` file do not exist.

  ```
  export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca-client/ca-admin
  rm -rf $FABRIC_CA_CLIENT_HOME/fabric-ca-client-config.yaml
  rm -rf $FABRIC_CA_CLIENT_HOME/msp
  ```
  {:codeblock}

5. Copy the TLS cert that you [downloaded from ICP](#ca-tls) to a directory where you can reference it in future commands. Ensure that you're in your `$HOME/fabric-ca-client` directory.

  ```
  cd $HOME/fabric-ca-client
  mkdir catls
  cp $HOME/tls.pem $HOME/fabric-ca-client/catls/tls.pem
  ```
  {:codeblock}

### Generating certificates with your CA admin
{: #enroll-ca-admin}

Before you deploy components or client applications with your CA, you need to generate certificates that authenticate you as an administrator with the ability to register new users. The process of generating the necessary certificates, your private key, and your public cert (also known as your enrollment cert or signCert), is called **enrollment**.

You can generate certificates only by using identities that have been registered with your Certificate Authority. Provide the identity's name and secret to generate certificates. A **CA admin** identity was automatically registered for you when you deployed your CA. You can now use that admin name and password to issue an `enroll` command with the Fabric CA client to generate an MSP folder with certificates that are then used to register other peer or orderer identities.

1. Ensure that you complete the steps to [set up the Fabric CA client](#fabric-ca-client) and that `$FABRIC_CA_CLIENT_HOME` is set to the directory where you want to store your CA admin certs.

  ```
  echo $FABRIC_CA_CLIENT_HOME
  $HOME/fabric-ca-client/ca-admin
  ```
  {:codeblock}

2. Run the following command to generate certificates:

  ```
  fabric-ca-client enroll -u https://<enroll_id>:<enroll_password>@<ca_url_with_port> --caname <ca_name>  --tls.certfiles  <ca_tls_cert_path>
  ```
  {:codeblock}

  The `<enroll_id>`and `<enroll_password>` in the command are the [CA admin user name and password](CA_deploy_icp.html#admin-secret) that you passed to the Kubernetes secret when you deployed the Certificate Authority. Insert the [CA URL](#ca-url) inside the `<ca_url_with_port>`. Leave off the `http://` at the beginning. The `<ca_name>` is the value that you provided to the `CA Name` field when you deployed the CA.

  The `<ca_tls_cert_path>` is the full path your [CA TLS cert](#ca-tls).

  A real call might look similar to the following example command:

  ```
  fabric-ca-client enroll -u https://admin:adminpw@9.30.94.174:30167 --caname org1CA --tls.certfiles $HOME/fabric-ca-client/catls/tls.pem
  ```

  The `enroll` command generates a complete set of certificates, which is known as a Membership Service Provider (MSP) folder, that is located inside the directory where you set to `$HOME` path for your Fabric CA client. For example, `$HOME/fabric-ca-client/ca-admin`. For more information about MSPs and what the MSP folder contains, see [Membership Service Providers](#msp).

  If the `enroll` command fails, see the [Troubleshooting topic](#ca-enroll-error) for possible causes.

You can run a tree command to verify that you have completed all of the prerequisite steps. Navigate to the directory where you stored your certificates. A tree command should generate a result similar to the following structure:

```
cd $HOME/fabric-ca-client
tree
.
├── ca-admin
│   ├── fabric-ca-client-config.yaml
│   └── msp
│       ├── cacerts
│       │   └── 9-30-250-70-30395-SampleOrgCA.pem
│       ├── keystore
│       │   └── 2a97952445b38a6e0a14db134645981b74a3f93992d9ddac54cb4b4e19cdf525_sk
│       ├── signcerts
│       │   └── cert.pem
│       └── user
└── catls
    └── tls.pem    
```

## Using the CA to deploy an orderer or peer
{: #deploy-orderer-peer}

Before you deploy an orderer or peer, you need to create a configuration JSON file that contains important information about the component identity and your CA. Then, you need to pass this file to the Helm chart during configuration by using a [Kubernetes Secret ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/concepts/configuration/secret/) object. This file allows your orderer or peer to get the certificates that it needs from the CA to join a blockchain network. This file also contains an admin certificate that allows you to operate your component as an admin user.

The following instructions provide you a [template JSON configuration file](#config-file-template) that you can edit and save to your local file system, and take you through how to use your CA to complete this file.

- Follow the instructions below if you are deploying an orderer on ICP or deploying a peer to connect to a consortium that is hosted on ICP.
- If you want to deploy a peer to connect to Starter Plan or Enterprise Plan, follow the instructions on [Deploying peers in IBM Cloud Private to connect to Starter Plan or Enterprise Plan](peer_deploy_ibp.html) instead. Those steps take you through how to use you CA on {{site.data.keyword.blockchainfull_notm}} Platform to deploy our peer on ICP.

### Configuration file
{: #config-file-template}

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

Copy this entire file into a text editor where you can edit it and save it to your local file system as a JSON file.

### CA connection information

First, we need to provide the connection information of your CA on ICP. Use the following information to complete values in the `"component"` section of the file:

- The `"cahost"` and `"caport"` values are the URL and port from the [CA URL](#ca-url). For example, if your CA URL is http://9.30.94.174:30167, the value of the `"cahost"` would be `9.30.94.174` and the `"caport"` would be `30167`.
- The `"caname"` is the name of the CA that was specified when you deploy the CA Helm chart. You referenced the CA name in your command when you enrolled by using the [CA admin identity](#enroll-admin).
- The `"cacert"` is the base64-encoded TLS certificate of your CA. [Retrieve your CA's TLS certificate](#ca-tls) from your ICP console by using the note in your Helm release, and insert the base64 output of the note commands in the following section.
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

- The fields in the `"tls"` section below require the same information as you passed to the component sections above, except you need to use the value of the CA TLS instance name that is specified during CA deployment. Use the same TLS cert for the `"cacert"` value.
  ```
  "tls": {
    "cahost": "",
    "caport": "",
    "caname": "",
    "catls": {
      "cacert": ""
  ```
  {:codeblock}

After retrieving your Certificate Authority connection information, you need to use the Fabric CA client to complete several operational steps:

1. [Register the new component with your CA](#register-component).

2. You also need to [register a component administrator](#register-admin) and then [generate certificates for the component admin](#enroll-admin) inside an MSP folder. You do not have to complete this step if you have already registered an admin to deploy another component.

3. [Register the new component with your TLS CA](#tls-register-component).

### Registering the component identity with the CA
{: #register-component}

If you want to found a consortium by deploying an ordering service and adding orgs to it, or to deploy peers and join them to channels, you first need to register the component identity with your CA. Your component deployment can then generate certificates that are necessary for the peer or orderer to participate in a network.

1. [Generate certificates with your CA admin](#enroll-ca-admin) by using the Fabric CA client. Use these admin certificates to issue the following commands. Ensure that `$FABRIC_CA_CLIENT_HOME` is set to `$HOME/fabric-ca-client/ca-admin`.

  ```
  echo $FABRIC_CA_CLIENT_HOME
  $HOME/fabric-ca-client/ca-admin
  ```

2. Issue the following command to find your affiliation and your organization name.
  ```
  fabric-ca-client affiliation list --caname <ca_name> --tls.certfiles <ca_tls_cert_path>
  ```
  {:codeblock}

  Your command might look like the following example:
  ```
  fabric-ca-client affiliation list --caname org1CA --tls.certfiles $HOME/fabric-ca-client/catls/tls.pem
  ```

  You should see information that is similar to the following example:
  ```
  affiliation: org1
      affiliation: org1.department1
  ```

  Make a note of the second **affiliation** value, for example, `org1.department1`. You need to use this value in the command below.

3. Run the following command to register the orderer or peer.

  ```
  fabric-ca-client register --caname <ca_name> --id.name <name> --id.affiliation org1.department1 --id.type <component_type> --id.secret <secret> --tls.certfiles <ca_tls_cert_path>
  ```
  {:codeblock}

  Create a name and password for the component and then use them to replace `name` and `secret`. **Important:** Make a note of this information. Set the `--id.type` to `orderer` if you are deploying an orderer, or set it to `peer` if you are deploying a peer. The command might look similar to the following example:

  ```
  fabric-ca-client register --caname org1CA --id.affiliation org1.department1 --id.name peer --id.secret peerpw --id.type peer --tls.certfiles $HOME/fabric-ca-client/catls/tls.pem
  ```

  After you run this command, you need to input the `name` and `secret` as the `"enrollid"` and `"enrollsecret"` in the configuration file, under the `"component"` section:

  ```
  "component": {...
    },
    "enrollid": "peer1",
    "enrollsecret": "peerpw",
  ```

  You can register an identity only once. If you experience a problem, try a command with a new user name and password. As a security measure, use each identity, and the accompanying enrollID and secret, to deploy only one peer. While you can use one **admin** identity for several components (this is our recommended deployment strategy), do not reuse peer IDs and passwords.

  When the command completes successfully, you should see information that is similar to the following example:

  ```
  2018/06/18 16:53:00 [INFO] Configuration file location: /fabric-ca-platform/admin/fabric-ca-client-config.yaml
  2018/06/18 16:53:00 [INFO] TLS Enabled
  2018/06/18 16:53:00 [INFO] TLS Enabled
  Password: peerpw
  ```

#### Registering the admin
{: #register-admin}

After you register the component, you also need to create an admin identity that you can use to operate the component. First, you need to register this new identity with your CA, and use it to generate an MSP folder. Then, you need to add the admin users signCert to the configuration file, where it will be made an admin cert of the orderer or peer during deployment. This allows you to use the certificates of the admin identity to operate your blockchain network, such as by starting a new channel or installing chaincode on your peers.

You need to create only one admin identity for the components that belong to your organization. If you are deploying multiple peers or an orderer, you need to complete these steps only once. You can use the signCert that you generated for one component to deploy any of the peers or orderer.

Ensure that your `$FABRIC_CA_CLIENT_HOME` is set to the path to the MSP of your CA Admin.

```
echo $FABRIC_CA_CLIENT_HOME
$HOME/fabric-ca-client/ca-admin
```
{:codeblock}

Register the component admin identity with the CA by running the following command:

```
fabric-ca-client register --caname <ca_name> --id.name <name> --id.affiliation org1.department1 --id.type user --id.secret <password> --tls.certfiles <ca_tls_cert_path>
```
{:codeblock}

Create a new user identity `name` and `secret` for the admin. This identity will be made the admin for all of the components that you deploy. Make sure to use different values than for the peer or orderer identity you just registered. The command looks similar to the following example.

```
fabric-ca-client register --caname org1CA --id.name peeradmin --id.affiliation org1.department1 --id.type user --id.secret peeradminpw --tls.certfiles $HOME/fabric-ca-client/catls/tls.pem
```

**Important:** Make a note of this information. You will use this `name` and `secret` to generate the MSP folder by using the `enroll` command.

#### Generating the admin MSP folder
{: #enroll-admin}

After your register the component admin, you can generate the certificates by using the following command:

```
fabric-ca-client enroll -u https://<peer_admin_enroll_id>:<peer_admin_enroll_password>@<ca_url_with_port> --caname <ca_name> -M $HOME/path/to/peer-admin/msp --tls.certfiles <ca_tls_cert_path>
```
{:codeblock}

For example:

```
fabric-ca-client enroll -u https://peeradmin:peeradminpw@9.30.94.174:30167 --caname org1CA -M $HOME/fabric-ca-client/peer-admin/msp --tls.certfiles $HOME/fabric-ca-client/catls/tls.pem
```

After this command completes successfully, it generates a new MSP folder in the directory that you specified by using the `-M` flag. This directory contains the certificates that you will use to operate your components. You can verify that the enroll command worked using a tree command. Navigate to the directory where you stored your certificates. A tree command should generate a result similar to the following structure:

```
cd $HOME/fabric-ca-client
tree
.
├── ca-admin
│   ├── fabric-ca-client-config.yaml
│   └── msp
│       ├── cacerts
│       │   └── 9-30-250-70-30395-SampleOrgCA.pem
│       ├── keystore
│       │   └── 2a97952445b38a6e0a14db134645981b74a3f93992d9ddac54cb4b4e19cdf525_sk
│       ├── signcerts
│       │   └── cert.pem
│       └── user
├── catls
│   └── tls.pem
├── orderer-admin
│   └── msp
│       ├── cacerts
│       │   └── 9-30-250-70-30395-SampleOrgCA.pem
│       ├── keystore
│       │   └── dfe06060490bf62e2bd709433fc747ff28cdbb1e040682c5d47a4e8598db4f2e_sk
│       ├── signcerts
│       │   └── cert.pem
│       └── user
├── peer-admin
│   └── msp
│       ├── cacerts
│       │   └── 9-30-250-70-30395-SampleOrgCA.pem
│       ├── keystore
│       │   └── 24f76217c5c7e641ee29b068712181294e427cbee4dbfce230a1a98b53489cbe_sk
│       ├── signcerts
│       │   └── cert.pem
│       └── user
└── tlsca-admin
    ├── fabric-ca-client-config.yaml
    └── msp
        ├── cacerts
        │   └── 9-30-250-70-30395-tlsca.pem
        ├── keystore
        │   └── 45a7838b1a91ddfe3d4d22a5a7f2639b868493bcce594af3e3ceb9c07899d117_sk
        ├── signcerts
        │   └── cert.pem
        └── user
```

In this MSP directory, open the signCert file of the new user and convert it to base64 format by using the following commands:

```
export FLAG=$(if [ "$(uname -s)" == "Linux" ]; then echo "-w 0"; else echo "-b 0"; fi)
cat $HOME/<path-to-peer-admin>/msp/signcerts/cert.pem | base64 $FLAG
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

For example:

```
export FLAG=$(if [ "$(uname -s)" == "Linux" ]; then echo "-w 0"; else echo "-b 0"; fi)
cat $HOME/fabric-ca-client/peer-admin/msp/signcerts/cert.pem | base64 $FLAG
```
{:codeblock}

This command prints out a string that is similar to the following example:

```
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNuRENDQWtPZ0F3SUJBZ0lVTXF5VDhUdnlwY3lYR2sxNXRRY3hxa1RpTG9Nd0NnWUlLb1pJemowRUF3SXcKYURFTTlEKaFhTTzRTWjJ2ZHBPL1NQZWtSRUNJQ3hjUmZVSWlkWHFYWGswUGN1OHF2aCtWSkhGeHBLUnQ3dStHZDMzalNSLwotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
```

Copy this string to the `"admincerts"` field under the component section in the configuration file.

#### Registering the component identity with the TLS CA
{: #tls-register-component}

You also need to register the orderer or peer with the TLS CA. To do this, you will first need to enroll using the admin of the TLS CA. Change `$FABRIC_CA_CLIENT_HOME` to a directory where you you want to store your TLS CA admin certificates.

```
cd $HOME/fabric-ca-client
mkdir tlsca-admin
export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca-client/tlsca-admin
```
{:codeblock}

Run the command below to enroll as your admin against the TLS CA. The command is the same as you used to enroll as your [CA admin](#enroll-ca-admin), only you will use the CA TLS instance name that you specified during your [CA configuration.](CA_deploy_icp.html#icp-ca-configuration-parms) Be sure to use the same `ca-admin-name` and `ca-admin-password` in your CA secret.

```
fabric-ca-client enroll -u https://<enroll_id>:<enroll_password>@<ca_url_with_port> --caname <tls_ca_name> --tls.certfiles <ca_tls_cert_path>
```
{:codeblock}

A real call might look similar to the following example:

```
fabric-ca-client enroll -u https://admin:adminpw@9.30.94.174:30167 --caname org1tlsca --tls.certfiles $HOME/fabric-ca-client/catls/tls.pem
```

After you have enrolled, you have the necessary certificates to register your component with the TLS CA. Run the following command to register the orderer or peer:

```
fabric-ca-client register --caname <ca_name> --id.name <name> --id.affiliation org1.department1 --id.type peer --id.secret <password> --tls.certfiles <ca_tls_cert_path>
```
{:codeblock}

This command is similar to the one that you used to register the component identity with the CA, except using the TLS CA name. If you are deploying an orderer instead of a peer, set `--id.type` to `orderer` instead of `peer`. You must give this identity a different user name and password then you used against your default CA. A real register might look similar to the command below:

```
fabric-ca-client register --caname tlsca --id.affiliation org1.department1 --id.name peertls --id.secret peertlspw --id.type peer --tls.certfiles $HOME/fabric-ca-client/catls/tls.pem
```

Copy the values of the `name` and `secret` to `"enrollid"` and `"enrollsecret"` under the `"tls"` section of the configuration file:

```
"tls": {...
  },
  "enrollid": "peertls",
  "enrollsecret": "peertlspw",
```

### CSR (Certificate Signing Request) hosts
{: #csr-hosts}

You need to provide a CSR hostname to deploy an orderer or peer. This hostname includes the proxy IP address of the cluster where you will deploy the component and the `service host name` that is generated when the Helm chart is deployed.

#### Locating the value of the cluster proxy IP address

If you want to deploy an orderer or peer on the same ICP cluster on which you deployed your CA, ensure that you have a [Cluster administrator ![External link icon](../images/external_link.svg "External link icon")](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/user_management/assign_role.html "Cluster administrator roles and actions") role on the ICP cluster where the orderer or peer will be deployed. Then, enter the same proxy IP that you used when you [configured for your CA](CA_deploy_icp.html#icp-ca-configuration-parms). If you want to deploy the orderer or peer on a different cluster, you can retrieve the value of the cluster proxy IP address from the ICP console by completing the following steps:

1. Log in to the ICP console. In the left navigation panel, click **Platform** and then **Nodes** to view the nodes that are defined in the cluster.
2. Click the node with the role `proxy` and then copy the value of the `Host IP` from the table.
3. Insert the `Host IP` as the value for `"hosts"` in the `"csr"` section of the configuration file below:

  ```
  "csr": {
    "hosts": [""]
  }
  ```
  {:codeblock}

#### Determining the service host name

A `service host name` is generated when a Helm chart is deployed. It is based on the `helm release name` that you use when you deploy the Helm chart.

- If you're deploying an orderer, the `service host name` is the `helm_release_name` that you specify during deployment with the string `-orderer` added at the end. For example, if your cluster IP proxy address is `9.42.134.44` and you specify a `helm release name` of `org1orderer`, you can insert the following values in the `"csr"` section of the file:

  ```
  "csr": {
    "hosts": [
       "9.42.134.44",
       "org1orderer-orderer"
     ]
  }
  ```
  {:codeblock}

- If you're deploying a peer, the `service host name` is the `helm release name` that you specify during deployment without a supplemental string. For example, if the cluster IP Proxy address is 9.42.134.44" and the `helm release name` is `org1peer1`, you can insert the following values in csr "hosts":

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

After you completed all the steps above, your updated configuration file might look similar to the following example:

```
{
    "enrollment": {
        "component": {
            "cahost": "9.30.20.70",
            "caport": "32129",
            "caname": "chandra46CA",
            "catls": {
                "cacert": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNGakNDQWIyZ0F3SUJBZ0lVTmlUbkdTandSdU1JVXhpWGcwMGZZWXhPSndJd0NnWUlLb1pJemowRUF3SXcKYURFTE1Ba0dBMVVFQmhNQ1ZWTXhGekFWQmdOVkJBZ1REazV2Y25Sb0lFTmhjbTlzYVc1aE1SUXdFZ1lEVlFRSwpFd3RJZVhCbGNteGxaR2RsY2pFUE1BMEdBMVVFQ3hNR1JtRmljbWxqTVJ0Z3WURWUVFERXhCbVlXSnlhV010ClkyRXRjMlZ5ZG1WeU1CNFhEVEU0TVRFeE5qRTJNVEF3TUZvWERUTXpNVEV4TWpFMk1UQXdNRm93YURFTE1Ba0cKQTFVRUJoTUNWVk14RnpBVkJnTlZCQWdURGs1dmNuUm9JRU5oY205c2FXNWhNUlF3RWdZRFZRUUtFd3RJZVhCbApjbXhsWkdkbGNqRVBNQTBHQTFVRUN4TUdSbUZpY21sak1Sa3dGd1lEVlFRREV4Qm1ZV0p5YVdNdFkyRXRjMlZ5CmRtVnlNRmt3RXdZSEtvWkl6ajBDQVFZSUtvWkl6ajBEQVFjRFFnQUU1dlBucDJUNTdkY2hTOGRLNExsMFJRZEEKd284RmJsMzBPcnBGdWFHUld5TFl4eGcxcVFTemhUY3hTcGtHZjh3a1FzVDVFb01lSWcrRytldjBOU01FUTZORgpNRU13RGdZRFZSMFBBUUgvQkFRREFnRUdNQklHQTFVZEV3RUIvd1FJTUFZQkFmOENBUUV3SFFZRFZSME9CQllFCkZMd2d1N0J3Uk9lQ2hzV2hWQWptMTdmalh1eVBNQW9HQ0NxR1NNNDlCQU1DQTBjQU1FUUNJR0FCRmNSdXhtSkIKY3c4OTJJOXhPS3YxVmYyT0JHZUh5N2pFQzRBRm5najFBaUJqdHFvdjBXMXdxZjhwcGttYkxIQkJoWW1vS3ZqRwo4bDQyeVQ5bWYxWVQrZz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K"
            },
            "enrollid": "peer",
            "enrollsecret": "peerpw",
            "admincerts": [
                "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMzRENDQW9PZ0F3SUJBZ0lVS2Vib0drZEJqenM5bllRbkVQTWp0VG56YTVrd0NnWUlLb1pJemowRUF3SXcKYURFTE1Ba0dBMVVFQmhNQ1ZWTXhGekFWQmdOVkJBZ1REazV2Y25Sb0lFTmhjbTlzYVc1aE1SUXdFZ1lEVlFRSwpFd3RJZVhCbGNteGxaR2RsY2pFUE1BMEdBMVVFQ3hNR1JtRmljbWxqTVJrd0Z3WURWUVFERXhCbVlXSnlhV010ClkyRXRjMlZ5ZG1WeU1CNFhEVEU0TVRFeE9URTNNRE13TUZvWERURTVNVEV4T1RFM01EZ3dNRm93ZmpFTE1BaKQTFVRUJoTUNWVk14RnpBVkJnTlZCQWdURGs1dmNuUm9JRU5oY205c2FXNWhNUlF3RWdZRFZRUUtFd3RJZVhCbApjbXhsWkdkbGNqRXVNQXNHQTFVRUN4TUVkWE5sY2pBTEJnTlZCQXNUQkc5eVp6RXdFZ1lEVlFRTEV3dGtaWEJoCmNuUnRaVzUwTVRFUU1BNEdBMVVFQXhNSFlXUnRhVzR4Y0RCWk1CTUdCeXFHU000OUFnRUdDQ3FHU000OUF3RUgKQTBJQUJLbjUwdEU5TmpZb0RFNDBqalh6RUJ2T2c3Y3RtOElRd0laMnRkS29iNEwwVVhKdSs1Tkt5S2dyUk9vbApWcjBmQmg5cWZWMjl4Nms0T2dmMFNiVklBZWlqZ2ZRd2dmRXdEZ1lEVlIwUEFRSC9CQVFEQWdlQU1Bd0dBMVVkCkV3RUIvd1FDTUFBd0hRWURWUjBPQkJZRUZOYWFkV0VzcGp2Smk1akpiVktiS2M3ZU1wUmhNQjhHQTFVZEl3UVkKTUJhQUZMd2d1N0J3Uk9lQ2hzV2hWQWptMTdmalh1eVBNQ2NHQTFVZEVRUWdNQjZDSEdOb1lXNWtjbUZ6TFcxaQpjQzV5WVd4bGFXZG9MbWxpYlM1amIyMHdhQVlJS2dNRUJRWUhDQUVFWEhzaVlYUjBjbk1pT25zaWFHWXVRV1ptCmFXeHBZWFJwYjI0aU9pSnZjbWN4TG1SbGNHRnlkRzFsYm5ReElpd2lhR1l1Ulc1eWIyeHNiV1Z1ZEVsRUlqb2kKWVdSdGFXNHhjQ0lzSW1obUxsUjVjR1VpT2lKMWMyVnlJbjE5TUFvR0NDcUdTTTQ5QkFNQ0EwY0FNRVFDSURGeApDYzE1bDZUZ1dqYnhSZzlmNjczOGV0K0NZZ1I3VVpGR200VHdoQk5jQWlBNmtUMFFwbDV6WnBUN3BzM0dySXlVCmEydDRHSTQ5ZTdjUm5PMmdrSzl6Z3c9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg=="
            ]
        },
        "tls": {
            "cahost": "9.30.20.70",
            "caport": "32129",
            "caname": "tlsca",
            "catls": {
                "cacert": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNGakNDQWIyZ0F3SUJBZ0lVTmlUbkdTandSdU1JVXhpWGcwMGZZWXhPSndJd0NnWUlLb1pJemowRUF3SXcKYURFTE1Ba0dBMVVFQmhNQ1ZWTXhGekFWQmdOVkJBZ1REazV2Y25Sb0lFTmhjbTlzYVc1aE1SUXdFZ1lEVlFRSwpFd3RJZVhCbGNteGxaR2RsY2pFUE1BMEdBMVVFQ3hNR1JtRmljbWxqTVJrd0Z3WURWUVFERXhCbVlXSnlhV010ClkyRXRjMlZ5ZG1WeU1CNFhEVEU0TVRFeE5qRTJNVEF3TUZvWERUTXpNVEV4TWpFMk1UQXdNRm93YURFTE1Ba0cKQTFVRUJoTUNWVk14RnpBVkJnTlZCQWdURGs1dmNuUm9JRU5oY205c2FXNWhNUlF3RWdZRFZRUUtFd3RJZVhCbApXhsWkdkbGNqRVBNQTBHQTFVRUN4TUdSbUZpY21sak1Sa3dGd1lEVlFRREV4Qm1ZV0p5YVdNdFkyRXRjMlZ5CmRtVnlNRmt3RXdZSEtvWkl6ajBDQVFZSUtvWkl6ajBEQVFjRFFnQUU1dlBucDJUNTdkY2hTOGRLNExsMFJRZEEKd284RmJsMzBPcnBGdWFHUld5TFl4eGcxcVFTemhUY3hTcGtHZjh3a1FzVDVFb01lSWcrRytldjBOU01FUTZORgpNRU13RGdZRFZSMFBBUUgvQkFRREFnRUdNQklHQTFVZEV3RUIvd1FJTUFZQkFmOENBUUV3SFFZRFZSME9CQllFCkZMd2d1N0J3Uk9lQ2hzV2hWQWptMTdmalh1eVBNQW9HQ0NxR1NNNDlCQU1DQTBjQU1FUUNJR0FCRmNSdXhtSkIKY3c4OTJJOXhPS3YxVmYyT0JHZUh5N2pFQzRBRm5najFBaUJqdHFvdjBXMXdxZjhwcGttYkxIQkJoWW1vS3ZqRwo4bDQyeVQ5bWYxWVQrZz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K"
            },
            "enrollid": "peertls",
            "enrollsecret": "peertlspw",
            "csr": {
                "hosts": [
                    "9.30.20.70",
                    "chandra48peer1"
                ]
            }
        }
    }
}
```
{:codeblock}

You can leave the other fields blank. Save this file and you need to use it when you deploy an [orderer](orderer_deploy_icp.html) or [peer](peer_deploy_icp.html).

## Membership Service Providers (MSPs)
{: #msp}

Components of {{site.data.keyword.blockchainfull_notm}} Platform consume identities via Membership Service Providers (MSPs). MSPs associate the certificates that the CAs issue with permissions and roles. For more information about MSPs, see [the Membership topic in the Hyperledger Fabric documentation ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/latest/membership/membership.html "the topic on Membership in the Hyperledger Fabric documentation").

MSP folders must have a defined structure to be used by Fabric components. The Fabric CA client establishes this structure by creating the following MSP folders:

- **cacerts:** Contains the certificate of the root CA of your network.
- **intermediatecerts:** These are the certificates of any intermediate CAs in your chain of trust (leading back to a root CA or CAs). Each Enterprise Plan organization has two intermediate CAs for failover and high availability.
- **signCerts:** This folder contains your public signing certificate, also known as your signCert or enrollment certificate. This certificate is attached to your calls to the network (a chaincode invoke, for example) when you reference your MSP directory from the command line or build a user context object with the SDKs. You can upload this certificate to IBP if you want to operate a network from the SDK or command line.
- **keystore:** This folder contains your private key. This key is used to sign calls to the network when you reference your MSP directory from the command line or build a user context object with the SDKs, but it is not attached to calls the way a signCert is. It is **crucial** that this key is kept safe. If it becomes compromised, it can be used to impersonate your identity.

You can also build an MSP folder that the fabric-ca-client can reference by using the Network Monitor and the Swagger APIs.

- **cacerts** and **intermediatecerts**: You can fetch these certificates with the [Swagger APIs](swagger_apis.html) by issuing a `Get` request to the MSP API.
- **signCerts** and **keystore**: You can generate these certificates by clicking the **Generate Certificates** button on the "Certificate Authority" panel. A pop-up window opens with two certificates listed. Copy and store the **Certificate** in signCerts and the **Private key** in keystore. Keep these certificates in a safe place because they are not stored on the platform.

Many Fabric components contain additional information inside their MSP folder. For example, if you operate a remote peer, you might see the following folders:

- **admincerts:** This folder contains the list of administrators for this organization or component. You need to upload your signCert to this folder if you are operating a remote peer from the command line or the SDKs. If you use the Fabric CA client, you also need an admincerts folder in your MSP containing the corresponding signCert to be recognized as administrator certs.
- **tls:** This is a folder where you store TLS certs that are used for communicating with other network components.

For more information about the structure of MSPs, see [Membership ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/latest/membership/membership.html "Membership") and [Membership Service Providers ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger-fabric.readthedocs.io/en/latest/msp.html "Membership Service Providers") in Hyperledger Fabric documentation.


## Viewing the CA logs
{: #ca-operate-view-logs}

Component logs can be viewed from the command line by using the [`kubectl CLI commands`](#ca-kubectl-configure) or through [Kibana ![External link icon](../images/external_link.svg "External link icon")](https://www.elastic.co/products/kibana "Your window into the Elastic Search"), which is included in your ICP cluster.

- Use the `kubectl logs` command to view the container logs inside the pod. If you are unsure of your pod name, run the following command to view your list of pods.

   ```
   kubectl get pods
   ```
   {:codeblock}

   Then, run the following command to retrieve the logs for the CA container that resides inside the pod by replacing `pod_name` with the name of your pod from the command output above:

   ```
   kubectl logs <pod_name> -c ca
   ```
   {:codeblock}

   For more information about the `kubectl logs` command, see [Kubernetes documentation ![External link icon](../images/external_link.svg "External link icon")](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs “Getting Started”)

- Alternatively, you can access deployment events and logs by using the [ICP cluster management console](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.0/troubleshoot/events.html), which opens the logs in Kibana.

  **Note:** When you view your logs in Kibana, you might receive the response `No results found`. This condition can occur if ICP uses your worker node IP address as its hostname. To resolve this problem, remove the filter that begins with `node.hostname.keyword` at the top of the panel and the logs will become visible.

## Troubleshooting
{: #ca-operate-troubleshooting}

### **Problem:** Error when running the `enroll` command
{: #ca-enroll-error}

When running the Fabric CA client enroll command, it is possible the command will fail with the following error:

```
Error: Failed to read config file at '/Users/chandra/fabric-ca-client/ca-admin/fabric-ca-client-config.yaml': While parsing config: yaml: line 42: mapping values are not allowed in this context
```
{:codeblock}

**Solution:**

This error can occur when your Fabric CA client tries to enroll but cannot connect to your CA. This can happen if:   

- Your `enroll` command contains an extra `https://` on the `-u` parameter.
- Your CA name is incorrect.
- Your username or password is incorrect.

Review the parameters you specified on your `enroll` command and ensure none of these conditions exist.
