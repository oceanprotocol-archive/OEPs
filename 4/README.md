```
shortname: 4/OAR
name: Ocean Assets Registry 
type: Standard
status: Raw
editor: Aitor Argomaniz <aitor@oceanprotocol.com>
contributors: Dimitri De Jonghe <dimi@oceanprotocol.com>, Mike Anderson <mike.anderson@dex.sg>, Samer Sallam <samer@oceanprotocol.com>, Kiran Karkera <kiran.karkera@dex.sg>
```



## Abstract

The Ocean Assets Registry OEP is a specification for Ocean Protocol to publish assets such as data sets, algorithms, infrastructure and so on.

This OEP does not focus on metadata structure, staking or curation. 
It's purely the mechanics of publishing and retrieval of assets. This OEP doesn't focus on Assets discovery. It will be related with the SONAR OEP.
Optionally this might contain a basic framework for the actor registry without the identity platform. 

## Motivation

Ocean network aims to power marketplaces for relevant AI-related data services. 
The data services require to be discoverable and queryable, as well as consumable.

ASSETS are DATA objects describing RESOURCES & SERVICES under control of a PUBLISHER.
A PUBLISHER is incentivized to PUBLISH ASSETS in order to make them discoverable for third parties.
A PROVIDER can discover ASSETS and give access to those ASSETS getting tokens in reward.

PUBLISHER publishes ASSET METADATA on OCEAN DB
CONSUMER queries OCEAN DB and finds ASSET METADATA
CONSUMER resolves PROVIDER for ASSET METADATA
CONSUMER creates ASSET SERVICE_AGREEMENT(token, proofs, ...) with PROVIDER
CONSUMER consumes ASSET SERVICE from PROVIDER

ASSETS metadata are STORED on a decentralized REGISTRY.
ASSETS metadata can be UPDATED
ASSETS metadata can be RETRIEVED from the REGISTRY.
ASSETS content can be RETRIEVED from the PROVIDER.
ASSETS can have a status of DISABLED, which implies that the ASSET cannot be CONSUMED anymore.

PROVIDER provides SERVICE and PROOF
VERIFIER validates PROOF


## Specification

The **Assets** information should be managed using an API. This API should exposes the following capabilities:

* Adding an Asset
* Get an Asset
* Update an Asset
* Disabling an Asset
* Make an Asset available through a Provider

The information or Metadata of the Assets should be registered on-chain.


### Proposed Solution

The proposed solution is composed of the following elements:
* A RESTful API exposing different methods implementing the different actions required. 
* A Keeper logic storing the Assets metadata information on-chain

The High Level API methods to implement from the **Ocean Agent** side are:

### Adding an Asset (ASE.001)

```
Path: /api/v1/keeper/assets/metadata
HTTP Verb: POST
Caller: Publisher
Input: Asset Schema
Output: Asset Schema
HTTP Output Status Codes: 
    HTTP 201 - Asset Created
    HTTP 422 - Asset already exists
```

![ASE.001](images/ASE.001.png "ASE.001")

This method implements the creation of an Asset. It includes a List of sample data included as reference with the Asset. Those samples should help the consumers to understand if the Asset includes the expected information.

This method is integrated by the **Publisher** of an Asset. Marketplaces can participate as proxies. 
The Input and Output parameters are defined in the [Asset Schema](https://github.com/oceanprotocol/pk-schemas/blob/develop/src/main/resources/avro/com/oceanprotocol/core/keeper/schemas/Asset.avsc).

### Get an Asset (ASE.002)

```
Path:  /api/v1/keeper/assets/metadata/{assetId}
HTTP Verb: GET
Caller: Any
Input: assetId
Output: Asset Schema
HTTP Output Status Codes: 
    HTTP 200 - OK
    HTTP 404 - Not Found
```

![ASE.002](images/ASE.002.png "ASE.002")


Method retrieving information (Metadata) of an Asset from the database. 
If the Asset retrieved has an attribute `contentState == “DISABLED”` the method should return a **HTTP 404** Not Found message.

This method could be integrated by any actor. 
The Input of this method is the assetId referencing to a unique Asset. The Output parameters are defined in 
the [Asset Schema](https://github.com/oceanprotocol/pk-schemas/blob/develop/src/main/resources/avro/com/oceanprotocol/core/keeper/schemas/Asset.avsc).


### Update an Asset (ASE.003)

```
Path: /api/v1/keeper/assets/metadata/{assetId}
HTTP Verb: PUT
Caller: Publisher
Input: Asset Schema
Output: Asset Schema
HTTP Output Status Codes: 
    HTTP 200 - Updated
    HTTP 404 - Not Found
```

![ASE.003](images/ASE.003.png "ASE.003")

This method implements the update of the information about an Asset. 

This method is integrated by the **Publisher** of an Asset. Marketplaces can participate as proxies. 
The Input and Output parameters are defined in the [Asset Schema](https://github.com/oceanprotocol/pk-schemas/blob/develop/src/main/resources/avro/com/oceanprotocol/core/keeper/schemas/Asset.avsc).


### Retering an Asset (ASE.004)

```
Path: /api/v1/keeper/assets/metadata/{assetId}
HTTP Verb: DELETE
Caller: Publisher
Input: assetId
Output: Asset Schema
HTTP Output Status Codes: 
    HTTP 200 - OK
    HTTP 404 - Not Found
```

![ASE.004](images/ASE.004.png "ASE.004")

This method implements a soft delete of an asset. It means the Asset is updated setting the contentState attribute to “DISABLED”. 
The method will return a **HTTP 200** status code and the Asset modified in the response body.

This method only can be integrated by the publisher of the Asset. 
The Input of this method is the assetId referencing to a unique Asset. The Output parameters are defined in 
the [Asset Schema](https://github.com/oceanprotocol/pk-schemas/blob/develop/src/main/resources/avro/com/oceanprotocol/core/keeper/schemas/Asset.avsc).



### Adding a Provider to an Asset (ASE.005)

```
Path: /api/v1/keeper/assets/provider
HTTP Verb: POST
Caller: Provider
Input: AssetProvider Schema
Output: AssetProvider Schema
HTTP Output Status Codes: 
    HTTP 201 - AssetProvider Created
    HTTP 400 - Request doesn’t include mandatory parameters
    HTTP 400 - Asset or Actor don’t exist in the system
    HTTP 422 - AssetProvider already exists
```

![ASE.005](images/ASE.005.png "ASE.005")

Method implementing the creation of a relation between an **Asset** and a **Provider**. 
This method is necessary when an Actor wants to be the Provider of an existing Asset. It will support different Providers providing the same asset from different Marketplaces. It should include the pricing & proof information about the asset to be provided by the actor.

This method is integrated by the **Provider** of an Asset. Marketplaces can participate as proxies. 
The Input and Output parameters are defined in the [AssetProvider Schema](https://github.com/oceanprotocol/pk-schemas/blob/develop/src/main/resources/avro/com/oceanprotocol/core/keeper/schemas/AssetProvider.avsc).



## Implementation

The implementation of this OEP is composed of the following developments:

* The implementation of the high level **Ocean Agent** API, it will expose the RESTful methods and will be done using Python and the Hug framework. 
* The keeper side will be implemented in the Decentralized VM component and Ocean Database.

### Assignee(s)
Primary assignee(s): @diminator, @ssallam, @shark8me


### Targeted Release

The implementation of this API is planned for the [Alpha release](https://github.com/oceanprotocol/ocean/milestone/1).


### Status
unstable


## Copyright Waiver
To the extent possible under law, the person who associated CC0 with this work has waived all copyright and related or neighboring rights to this work.

