# OEP-14: Ocean Commons Marketplace

```
shortname: 14/COMMONS-MKT
name: Ocean Commons Marketplace
type: Standard
status: Raw
editor: Aitor Argomaniz <aitor@oceanprotocol.com>
contributors: Matthias Kretschmann <matthias@oceanprotocol.com>
```

Table of Contents
=================

- [Motivation](#motivation)
- [Actors](#actors)
- [Flow](#flow)
  - [Publishing](#publishing)
    - [DDO specifics](#ddo-specifics)
  - [Importing from Google Dataset](#importing-from-google-dataset)
  - [Consuming](#consuming)
  - [User Interface Screens](#user-interface-screens)
- [Next steps](#next-steps)


-----

The intention of this document is to describe the scenario of a working Ocean Commons Marketplace. There the users should be able to publish and discover free/commons assets (datasets). 

It's out of the scope to detail the implementation of components (can be found in the [OEP-11](https://github.com/oceanprotocol/OEPs/tree/master/11)

## Motivation

The motivations of this use case are:

* Get contents. Populate of Commons/Free datasets the first live environment release of Ocean
* Provide a end to end scenario where users can discover and get access to publicly available datasets
* Implement and Validate the network rewards function of the Ocean network in the case of free assets
* Integrate all the components and validate the architecture


## Actors

The different actors interacting in this flow are:

* **PUBLISHER** - The user discovering a public available Asset on the internet.
  The publisher could be an Ocean Ambassador or other user publishing the
  Metadata and links to the assets using the Ocean Commons Marketplace.
  The main incentive of the Publisher is to get rewards.
* **PROVIDER** - Is the entity running the Marketplace software allowing to publish and consume assets.
  Ocean Protocol Foundation will be the entity running all the provider capabilities. It includes the following components:
  - **COMMONS MARKETPLACE** (aka **MKT**) - Frontend application where users can publish and discover commons/free datasets
  - **AQUARIUS** - Metadata API storing the commons datasets metadata
  - **BRIZO** - Proxy in charge of validate the Service Execution Agreements (SEA)
* **KEEPER** - Distributed ledger running the Solidity Smart Contracts on Ethereum VM.
  It includes the Ocean Token, escrow payment function and SEA logic.
* **CONSUMER** - A user, typically a data scientist or data engineer, looking for new data to use in their data pipelines.


## Flow

### Publishing

The Publishing flow describe the procedure to put in place allowing to an Ocean Protocol PUBLISHER to register new commons assets in the OCEAN COMMONS MARKETPLACE and get a reward for it. The complete flow is the following:

1. The PUBLISHER, search on the internet and discover a relevant free/commons asset, get the publicly available information of the asset, metadata and license information.

2. The PUBLISHER goes to the Ocean Commons Marketplace frontend application, and open the form of "Register a new Commons/Free Asset"

3. The PUBLISHER fill the following information:
   - List of public URL's where the Asset is located
   - Title
   - Description
   - Category (categories can be predefined upfront)
   - Author
   - Tags
   
   A complete list of attributes can be found in the [OEP-8](https://github.com/oceanprotocol/OEPs/tree/master/8).
   
4. The MKT validate that URL's are correct and contents are available in those URL's initiating the download of the contents. Contents are not persisted in any place, the MKT application only validate that content exist in the URL's provided. If any URL is invalid, the processed is cancelled and the user is informed of the error.

5. The MKT validate the input fields provided by the user. If information is missing, the process is cancelled and the user is informed of the error.

6. The MKT, using SQUID-JS does the following:
   - Create a new DDO for the Asset
   - Encrypt the URL's using the PROVIDER Keys
   - Add the encrypted URL's and Metadata to the DDO
   - The PUBLISHER signature is added to the DDO in the **publicKey** section. Type `OwnerAddress`
   - The PROVIDER signature is added to the DDO in the **publicKey** section. Type `ProviderAddress`
   - The DDO should include the prize equals to zero in the conditions for the access service.
   - Persist the DDO in the Provider AQUARIUS instance

7. The MKT, using SQUID-JS registers the DID on-chain associating the public URL where the DDO just published is available

#### DDO specifics

- The `publicKey` section includes the Owner and Provider addresses:

```
{
  ...

  "publicKey": [{
    "id": "did:op:123456789abcdefghi",
    "type": "OwnerAddress",
    "owner": "0x12345fwgrebrewfewfew"
  }, {
    "id": "did:op:0987654321abcdefghi",
    "type": "ProviderAddress",
    "owner": "0x12345fwgrebrewfewfew"
  }

  ...
}
```

- The price in the Access conditions is zero


### Importing from Google Dataset

This flow is similar to the previous one. The main intention of this flow is to re-use the existing contents in Google Dataset service making them available in the Commons Marketplace.

The main differences of this flow are:

1. The PUBLISHER, search on [Google Datasets](https://toolbox.google.com/datasetsearch/) website  and discover a relevant free/commons asset

1. The PUBLISHER goes to the Ocean Commons Marketplace frontend application, and open the form of "Import a Commons/Free Asset from Google Datasets"

1. The PUBLISHER fill the following information:
   - URL of the Google Dataset to import (i.e: https://toolbox.google.com/datasetsearch/search?query=Weather%20in%20Australia&docid=gEf986q6CT26lj9yAAAAAA%3D%3D)
   - List of URL's of the contents associated (can't be imported directly)

1. The MKT app retrieve the metadata from the Google Dataset URL

1. The MKT app validate that all contents provided in the URL's are available 

The rest of the flow is the same to the one described previosly.


### Consuming

The Consuming flow describe the procedure to put in place allowing to an Ocean Protocol CONSUMER (typically a data engineer or data scientist) to discover and get access to free/commons assets using the OCEAN COMMONS MARKETPLACE.

The complete flow is the following:

1. The CONSUMER using the MKT application write a search query

1. The MKT send the search query to AQUARIUS to retrieve the list of results (pending to define the ranking)

1. The MKT frontend application draw the list of results showing the most relevant information (title, tags, category, etc)

1. The CONSUMER select one of the ASSETS and open the **detailed view page** using the MKT app

1. The MKT app draw in the **detailed view page** all the metadata associated to the asset. The asset URL must be unique, and include the DID in the URL. Example: 
   `http://commons.oceanprotocol.com/asset/did:op:4d517500da0acb0d65a716f61330969334630363ce4a6a9d39691026ac7908ea`

   Also should include all the metadata tagged in the page allowing an easy SEO for the contents.

1. The **detailed view page** shows a button or download link for each of the content urls existing in the DDO. 
 
1. When the CONSUMER clicks on the "DOWNLOAD" link of any of the contents, the CONSUMER using SQUID-JS initialize the purchase process.

1. SQUID-JS calls to BRIZO to initialize the Service Executing Agreement (SEA)

1. BRIZO initialize the SEA on-chain using the KEEPER

1. The KEEPER after initialization emits the `AgreementCreated` and `AgreementActorAdded` event

1. The CONSUMER listens for the `AgreementCreated` and `AgreementActorAdded` event and when it's received call the `lockPayment` function on the KEEPER

1. The `lockPayment` function on the KEEPER emit the `PaymentLocked` event

1. BRIZO listens for the `PaymentLocked` event and when it's received call the `GrantAccess` function in the KEEPER

1. The Keeper emit the `AccessGranted` event

1. The CONSUMER listens for the `AccessGranted` event and when it's received call BRIZO sending the encrypted URL

1. BRIZO valide on-chain using the `checkPermissions` function in the KEEPER Access Conditions if the user has access. If not cancel the request returning a `HTTP 401 Unathorized` answer.

1. BRIZO, working as a PROXY, return the Asset content to the CONSUMER

The complete PUBLISH and CONSUME flows can be shown in the below image:

![Publish and Consume flow](board.jpg)

### User Interface Screens

In a high-level the Commons Marketplace interface is composed by the following screens:

- The HOME page. The main component here is the search box. It allows to run search queries on top of the AQUARIUS database.
  The HOME page also could include some additional information about what is the common marketplace, pre-defined queries, most relevant assets, etc.

- The LIST of results. When the user search for an asset, this page will show the list of results.
  Should include brief information about the asset, like title, tags, etc. If the user click in the Asset in the LIST will be redirected to the DETAILS PAGE.

- The DETAILS page. It includes the complete information about the Asset and the DOWNLOAD button.

- The PUBLISH page. It includes the form to add the Metadata and the content urls.


## Next steps

Further releases of this could be:

* Curation - Assets can be rated (stars, thumb up/down, etc). This rating would be used to rank the search results in the List page.

* Staking - Publishers have the possibility to stake in the Assets they publish. Depending of the stake and the reputation/curation/validation of the Asset, the Publishers could have a higher reward because the stake or lose it if they publish invalid contents.

* Orders - Users publishing assets could access to a page with the list of assets published and some usage stats (e.g number of downloads). Consumers could have a similar page with the Assets the purchased/downloaded.

* Reviews - In the Detailed view, the users could add their reviews about the asset. Those reviews would be shown in the same page.
