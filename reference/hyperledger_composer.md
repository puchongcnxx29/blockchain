---

copyright:
  years: 2017, 2018
lastupdated: "2018-12-08"
---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}


# Hyperledger Composer
{: #hyperledger-composer}

**IBM recommends using Hyperledger Composer solely for demos and proof-of-concepts. IBM does not provide support for networks using Hyperledger Composer in production, including the Composer CLI, JavaScript APIs, REST server, and Web Playground.**

***[Is this page helpful? Tell us.](https://www.surveygizmo.com/s3/4501493/IBM-Blockchain-Documentation)***


{{site.data.keyword.blockchainfull}} Platform: Develop is built on the open-source Hyperledger Composer toolset, one of the projects within the Linux Foundation's Hyperledger Project. Hyperledger Composer enables architects and developers to quickly create {{site.data.keyword.blockchain}} solutions with REST APIs that expose the business logic to web or mobile applications, as well as integrating blockchain with existing enterprise systems of record.
{:shortdesc}

Hyperledger Composer is composed of a runtime, a command line interface, a REST server, LoopBack connector, Playground user interface, Yeoman code generator, and VSCode and Atom editor plugins. For more information on Hyperledger Composer, see the [Hyperledger Composer documentation ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger.github.io/composer/latest/introduction/introduction.html)


## Business networks

Hyperledger Composer allows you to quickly model your current business network, which contains your existing assets and the transactions related to them; assets are tangible or intangible goods, services, or property. As part of your business network model, you define the transactions which can interact with assets. Business networks also include the participants who interact with them, each of which can be associated with a unique identity, across multiple business networks.

For more information on building business networks see [Developing business networks](../develop.html) or the [Hyperledger Composer documentation ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger.github.io/composer/latest/introduction/introduction.html).

## Queries

Queries are used to return data about the blockchain world-state. Queries are defined within a business network, and can include variable parameters for simple customization. By using queries, data can be easily extracted from your {{site.data.keyword.blockchain}} network. Queries are sent by using the [Hyperledger Composer API ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger.github.io/composer/latest/api/api-doc-index).

For more information about using queries in your business network, see [Querying and filtering business network data ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger.github.io/composer/latest/tutorials/queries).

## Identities and business network cards

Within a business network, participants are associated with an identity. Business network cards are a combination of an identity, a connection profile, and metadata; and are used to connect to a business network and demonstrate identity for transactions. Business network cards simplify the process of connecting to a business network, and extend the concept of an identity outside the business network to a 'wallet' of identities, each associated with a specific business network and connection profile.

For more information about identities and business network cards, see [Participants and identities ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger.github.io/composer/latest/managing/participantsandidentities).

## Hyperledger Composer REST server

The Hyperledger Composer REST server automatically generates a Swagger REST API for a business network. The REST server is based on LoopBack and converts the business network model into an Open API definition. At runtime the REST server implements Create, Read, Update, and Delete support for assets and participants and allows transactions to be submitted for processing or retrieved with queries.

For more information about the Hyperledger Composer REST server, see [Generating a REST API ![External link icon](../images/external_link.svg "External link icon")](https://hyperledger.github.io/composer/latest/integrating/getting-started-rest-api).
