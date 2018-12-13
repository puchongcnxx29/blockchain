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
{:tip: .tip}

# Deploying a business network on Starter Plan
{: #deploying-a-business-network}

**IBM recommends using Hyperledger Composer solely for demos and proof-of-concepts. IBM does not provide support for networks using Hyperledger Composer in production, including the Composer CLI, JavaScript APIs, REST server, and Web Playground.**

*[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)*


Business networks can be developed and deployed to a Starter Plan environment by using the {{site.data.keyword.blockchainfull}} Platform developer environment and Hyperledger Composer developer toolset.
{:shortdesc}

By using the developer environment, you can quickly model and test {{site.data.keyword.blockchain}} business networks and deploy them to an instance of the {{site.data.keyword.blockchainfull_notm}} Platform.

## Before you begin

Ensure that you read the [About Starter Plan](./starter_plan.html) and [Getting started with Starter Plan](./get_start_starter_plan.html). Also ensure that you have installed the [{{site.data.keyword.blockchainfull_notm}} Platform developer environment](./develop_install.html) and created an instance of the {{site.data.keyword.blockchainfull_notm}} Platform Starter Plan following the instructions in [Govern Starter Plan network](./get_start_starter_plan.html).

Ensure you have Node v8.9 or higher, npm v5.x, and Hyperledger Composer:

- If your network is on Fabric version 1.2, use Hyperledger Composer v0.20.x.
- If your network is on Fabric version 1.1, use Hyperledger Composer v0.19.x.  

You can find your Fabric version by opening the [Network Preferences window](../v10_dashboard.html#network-preferences) in your Network Monitor.


## Step One: Retrieve admin secret

1. From the Starter Plan overview screen, click **Connection Profile** and then download. Rename this file to 'connection-profile.json'.

2. Move this file to be in the same directory as your `.bna` file.

3. Inside the connection profile, go all the way down until you see 'registrar'. Inside 'registrar', under 'enrollId' there is an **enrollSecret** property. Retrieve the secret and save a copy of it.

    ![Retrieve admin secret](images/get_enroll_secret.gif "Retrieve admin secret")


## Step Two: Creating a certificate authority card

The secret retrieved in the preceding step will be used to create a business network card for the certificate authority (CA). The CA card will then be imported and the card will be used to exchange the **enrollSecret** for valid certificates from the Starter Plan certificate authority.

1. Using the **enrollSecret** noted from step one, run the following command to create the CA business network card:

   ```
   composer card create -f ca.card -p connection-profile.json -u admin -s enrollSecret
   ```
   {:pre}

Replace `enrollSecret` in the preceding command with the admin secret retrieved from the connection profile.

2. Import the card using the following command:

   ```
   composer card import -f ca.card -c ca
   ```
   {:codeblock}

3. Now that the card is imported, it can be used to exchange the **enrollSecret** for valid certificates from the CA. Run the following command to request certificates from the certificate authority:

   ```
   composer identity request --card ca --path ./credentials -u admin -s enrollSecret
   ```
   {:codeblock}

Replace `enrollSecret` in the preceding command with the admin secret retrieved from the connection profile. The `composer identity request` command creates a `credentials` directory that contains certificate `.pem` files.

## Step Three: Adding the certificates to the Starter Plan instance

The certificates must be added to the Starter Plan instance. For convenience, they can be added by using the {{site.data.keyword.blockchainfull_notm}} Platform UI. The certificates must be added, then the peers must be restarted, and then the certificates must be synced on the channel. The certificate needed is the `admin-pub.pem` file that was generated from the previous command, which is in the `credentials` directory.

1. In the Starter Plan UI, click on the **Members** tab, then **Certificates**, then **Add Certificate**. Go to your `credentials` directory, and copy and paste the contents of the `admin-pub.pem` file in the certificate box. Submit the certificate and restart the peers. Note: restarting the peers takes a minute.

    ![Add certificates](images/add_cert.gif "Add certificates")

2. Next, the certificates must be synced on the channel. Click the **Channels** tab, then the **Actions** button, then **Sync Certificate** and **Submit**.

    ![Sync certificates](images/sync_cert.gif "Sync certificates")

## Step Four: Creating an admin business network card

Now that the correct certificates have been synced with the peers, business network cards can be created which have the permissions to install the Hyperledger Composer runtime and start chaincode.

1. Create an admin card with the channel admin and peer admin roles by using the following command:

   ```
   composer card create -f adminCard.card -p connection-profile.json -u admin -c ./credentials/admin-pub.pem -k ./credentials/admin-priv.pem --role PeerAdmin --role ChannelAdmin
   ```
   {:codeblock}

   This card will be used to deploy a business network to Starter Plan.

2. Import the card created in the previous step using the following command:

   ```
   composer card import -f adminCard.card -c adminCard
   ```
   {:codeblock}

## Step Five: Installing and starting the business network

Next, the card created in the previous step can be used to install and start a business network. For this guide, we'll install the vehicle manufacturing network sample, either use the vehicle manufacturing sample or install your own business network, but make sure to change the business network names specified in the commands. The command to start a business network will also create a card. For Starter Plan, this card must be deleted, the example command given names this card `delete_me.card` so it can be easily distinguished.

1. Install the Hyperledger Composer runtime with the following command:

   ```
   composer network install -c adminCard -a vehicle-manufacture-network.bna
   ```
   {:codeblock}

   Note the business network version number which is returned when you run this command. It will be required in the next step.

2. Start the business network with the command below. If you get an error, wait a minute and try again. Use the version number from the last step after the `-V` option.

    ```
    composer network start -c adminCard -n vehicle-manufacture-network -V 0.0.1 -A admin -C ./credentials/admin-pub.pem -f delete_me.card
    ```
    {:codeblock}

3. Delete the business network card called `delete_me.card`.

4. Create a new business network card and reference the certificates that are retrieved earlier with the following command:

   ```
   composer card create -n vehicle-manufacture-network -p connection-profile.json -u admin -c ./credentials/admin-pub.pem -k ./credentials/admin-priv.pem
   ```
   {:codeblock}

5. Import the business network card with the following command:

    ```
    composer card import -f ./admin@vehicle-manufacture-network.card
    ```
    {:codeblock}

The business network is now deployed to the Starter Plan instance.

## Step Six: Ping the business network to ensure it is running correctly

Run the following command to ping the business network:

   ```
   composer network ping -c admin@vehicle-manufacture-network
   ```
   {:codeblock}

To view the chaincode logs, click **Channels**, and then select your channel. <!-- Click the dropdown arrow to view the logs, or the Actions symbol to view in more detail. --> Click the **Chaincode** tab. Expend the chaincode row and then click the **JSON** or **Logs** button.

<!-- [fN-Yuj](https://i.makeagif.com/media/4-13-2018/fN-Yuj.gif) -->
