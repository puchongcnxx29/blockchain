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

# Installing a business network development environment
{: #installing-a-development-environment}

**IBM recommends using Hyperledger Composer solely for demos and proof-of-concepts. IBM does not provide support for networks using Hyperledger Composer in production, including the Composer CLI, JavaScript APIs, REST server, and Web Playground.**

***[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


Follow these instructions to obtain the {{site.data.keyword.blockchainfull}} Platform: Develop development tools for creating and testing business networks. To provide flexibility and to enable the maximum number of dev, test, and deployment scenarios, the development tools are delivered as npm modules, which are controlled from the command line.

## Before you begin

Ensure that you install the following prerequisites:

- Ubuntu Linux 14.04 / 16.04 LTS (both 64-bit), or Mac OS 10.12
- Node v8.9 or higher (v9 is not supported)
- npm v5.x
- git v2.9.x or higher
- Python v2.7.x
- A code editor of your choice, like VSCode.

If you are using Enterprise Plan, install the following additional prerequisites:

- Docker Engine v17.03 or higher
- Docker-Compose v1.8 of higher

If you are using VSCode, there is a Hyperledger Composer VSCode extension available [here](https://marketplace.visualstudio.com/items?itemName=HyperledgerComposer.composer-support-client).

## Step One: Install the command line tools

The command line tools include **composer-cli**, the main {{site.data.keyword.blockchainfull_notm}} Platform: Develop command line module; **generator-hyperledger-composer**, **composer-rest-server**, and **Yeoman**, these modules are used to generate business networks, bespoke REST servers, and Angular applications. The version you install depends on the version of Hyperledger Fabric that your Starter or Enterprise plan is running. You can find your Fabric version by opening the [Network Preferences window](../v10_dashboard.html#network-preferences) in your Network Monitor.

1. For use with new instances of Starter Plan running Hyperledger Fabric v1.2.1, install **composer-cli** by using the following command:

    ```
    npm install -g composer-cli@0.20.x
    ```
    {:codeblock}

  For use with Enterprise Plan and Starter Plan instances running Hyperledger Fabric v1.1, install **composer-cli** by using the following command:

    ```
    npm install -g composer-cli@0.19.x
    ```
    {:codeblock}

2. For use with new instances of Starter Plan running Hyperledger Fabric v1.2.1, install **composer-rest-server** by using the following command:

    ```
    npm install -g composer-rest-server@0.20.x
    ```
    {:codeblock}

  For use with Enterprise Plan and Starter Plan instances running Hyperledger Fabric v1.1, install **composer-rest-server** by using the following command:

    ```
    npm install -g composer-rest-server@0.19.x
    ```
    {:codeblock}

 Use the **composer-rest-server** module to create a REST Server on your machine to expose your business networks as RESTful APIs.

3. For use with new instances of Starter Plan running Hyperledger Fabric v1.2.1, install **generator-hyperledger-composer** by using the following command:

    ```
    npm install -g generator-hyperledger-composer@0.20.x
    ```
    {:codeblock}

  For use with Enterprise Plan and Starter Plan instances running Hyperledger Fabric v1.1, install **generator-hyperledger-composer** by using the following command:

    ```
    npm install -g generator-hyperledger-composer@0.19.x
    ```
    {:codeblock}

    Use **generator-hyperledger-composer** to generate skeleton business network structures, models, and an Angular application.

4. Yeoman is a tool for generating applications, which utilizes `generator-hyperledger-composer`:

    ```
    npm install -g yo
    ```
    {:codeblock}

## Step Two: Install a local Playground

The Hyperledger Composer playground is a user interface that can be used as a simulated environment for testing a business network. Install the playground by using the following command:

```
npm install -g composer-playground@0.20.x
```
{:codeblock}


## Optional: Set up your IDE

You can use the playground to develop, edit, and test business networks. However, if you want to develop a business network in an IDE, there is a VSCode extension available, which adds syntax highlighting for the Hyperledger Composer modelling language.

1. Install VSCode from this URL: [https://code.visualstudio.com/download](https://code.visualstudio.com/download)

2. Open VSCode, go to Extensions, then search for and install the Hyperledger Composer extension from the marketplace.


### Starting and stopping Hyperledger Fabric

There are a set of scripts in the `fabric-tools` directory, which control the Hyperledger Fabric instance. You can stop and start the runtime by using `~/fabric-tools/stopFabric.sh` and `~/fabric-tools/startFabric.sh`.

The `~/fabric-tools/teardownFabric.sh` will require you to create a new PeerAdmin card when the environment is next started.
