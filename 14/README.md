```
shortname: 14/OAR
name: Ocean Assets Registry
type: Standard
status: Raw
editor: Aitor Argomaniz <aitor@oceanprotocol.com>
contributors: Dimitri De Jonghe <dimi@oceanprotocol.com>
              Fang Gong <fang@oceanprotocol.com>
              Ahmed Ali <ahmed@oceanprotocol.com>
```

<!--ts-->

Table of Contents
=================

   * [Table of Contents](#table-of-contents)
   * [Ocean Assets Registry](#ocean-assets-registry)
      * [Change Process](#change-process)
      * [Language](#language)
      * [Motivation](#motivation)
      * [Proposed Solution](#proposed-solution)
      * [Asset Lifecycle](#asset-lifecycle)
         * [Registering a new Asset](#registering-a-new-asset)
         * [Retrieve Metadata of an Asset](#retrieve-metadata-of-an-asset)
         * [Updating Asset Metadata](#updating-asset-metadata)
         * [Retiring an Asset](#retiring-an-asset)
      * [Copyright Waiver](#copyright-waiver)
      

      
<!--te-->

<a name="ocean-assets-registry"></a>
# Ocean Assets Registry

The Ocean Assets Registry (**OAR**) is a specification for Ocean Protocol to register any kind of Data Asset in the Ocean Network. 
In the scope of Ocean, we understand as **Asset** or **Data Asset** as any kind of data stored in a structured machine readable format.
It could includes Datasets or Algorithms.

This OEP does not focus on metadata structure, staking or curation. It's purely the mechanics of registering and basic handling of assets.
This OEP discuss about the Data Assets metadata, and not about the Assets storage and consumption mechanisms.
This OEP doesn't focus on Assets discovery. It will be related with the SONAR OEP.

This specification is based on [Ocean Protocol technical whitepaper](https://github.com/oceanprotocol/whitepaper), [3/ARCH](../3/README.md), [4/KEEPER](../4/README.md) and [5/AGENT](../5/README.md).


<a name="change-process"></a>
## Change Process
This document is governed by the [2/COSS](../2/README.md) (COSS).

<a name="language"></a>
## Language
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.

<a name="motivation"></a>
## Motivation

Ocean network aims to power marketplaces for relevant AI-related data services.
The data assets require to be registered in the system and managed in a basic way.  

Requirements are:

* ASSETS are DATA objects describing RESOURCES under control of a PUBLISHER:
* PUBLISHERS are incentivized to PUBLISH ASSETS in order to make them discoverable for third parties
* PROVIDER can give access to some ASSETS getting tokens in reward
* PUBLISHER could publish ASSET METADATA on OCEAN DB or an independent Database
* CONSUMER queries OCEAN DB or an independent Database and find ASSETS METADATA 
* CONSUMER resolves PROVIDER for ASSET METADATA 
* CONSUMER creates ASSET SERVICE_AGREEMENT(token, proofs, ...) with PROVIDER 
* CONSUMER consumes ASSET SERVICE from PROVIDER
* ASSET essential attributes are STORED on a decentralized REGISTRY 
* ASSET METADATA is stored in OCEAN DB if it's enabled in the AGENT
* ASSET metadata can be RETRIEVED from OCEAN DB if it's enabled in the AGENT
* ASSET content can be RETRIEVED from the PROVIDER
* ASSETS can have a status of DISABLED or RETIRED, which implies that the ASSET cannot be CONSUMED anymore
* PROVIDER provides SERVICE and PROOF VERIFIER validates PROOF
* OCEAN DB is optional/pluggable, OAR MUST work without an OCEAN DB backend 
  

## Proposed Solution 

The **ASSET** information should be managed using an API. 
As general rule, only the **INDISPENSABLE** information to run the Smart Contracts MUST be stored in te Keeper VM.
This API should exposes the following capabilities:

* Registering a new Asset
* Retrieve metadata information of an Asset
* Update the metadata of an existing Asset 
* Retire an Asset
* Make an Asset available through a Provider

The following restrictions apply during the design/implementation of this OEP:

* The Assets registered in the system MUST be associated to the Actors registering the Assets
* The Actors associated to the Assets (PUBLISHER or PROVIDER) MUST have a valid Account Id in the system
* The information or Metadata about the Assets will be stored in Ocean DB if the user plugs a valid Ocean DB implementation
* Only the very basic information about the Assets (ids and pricing) MUST be stored in the Keeper VM too 
* AGENT MUST NOT store any information about the Assets or Actors during this process



The proposed solution is composed by the interaction of different elements:

* A high level HTTP API exposing the methods required to manage the Assets Registry (AGENT)
* A Keeper node registering the complete Assets metadata (KEEPER - Ocean DB). This is optional and depends of the user parameters.
* A Keeper node registering the Asset IDs (KEEPER VM)
* A backend orchestration layer (AGENT) in charge of coordinating the persistence of the Assets in both backends consistently (Ocean DB & Keeper VM)

We can show the interaction between layers and components using a stack view:

![stack view](images/arch-assets-stack.png)

Main interactions involving PUBLISHERS, PROVIDERS and Ocean nodes are using the Keeper nodes as source of truth. 
In the below image you can see an example of interaction:

![high level interactions](images/arch-assets-interactions2.png)

The above diagram shows the high level interactions between the components involved:

* The PUBLISHER interacting with the AGENT sending a request to register the Metadata of a new ASSET
* The AGENT MUST validate the basic parameters sent by the PUBLISHER
* The AGENT MUST authenticate the PUBLISHER sending the request
* The AGENT MUST authorize the user via KEEPER
* The AGENT MUST orchestrate the Asset registering in the OCEAN DB and Keeper VM
* The Keeper VM MUST only store as less information as possible. Only the main IDs and pricing information
* The OCEAN DB MUST store the complete ASSET Metadata if it's configured/instantiated (this is OPTIONAL)
* The AGENT MUST validate the basic parameters sent by the PUBLISHER
* The AGENT MUST authenticate the PUBLISHER sending the request
* The AGENT MUST authorize the user via KEEPER
* The OCEAN DB MUST store the relation between the ASSET and the PROVIDER if it's configured/instantiated (this is OPTIONAL)


In the below diagram you can see an alternative interaction where the Marketplace is not involved: 

![high level interactions](images/arch-assets-interactions-nomkt.png)

The following sections will describe the end to end implementation using a top to bottom approach, 
starting from the API interface to the Keeper implementation, using the Ocean DB and the Keeper VM.

In the following diagram you can see the nodes involved in this OEP:

![Nodes Involved](images/nodes-involved1.png "Nodes Involved")

In the following sections you can find the end to end implementation details of the complete OAR functionality.

### Smart Contracts

The KEEPER will store only the essential user information to allow the implementation of the Assets Registry. It means from the KEEPER side, the system MUST NOT store any kind of metadata.

Taking this into account, the skeleton of main implementation should provide the following structs and interfaces:

```solidity
contract AssetsRegistry {

    struct Asset {
        address owner;
        uint256 price;
        bool active;
    }
    
    // associate asset id to the Asset struct
    mapping(bytes32 => Asset) public mAssets; 
    
    /////// EVENTS //////////////////////////////
    event AssetRegistered(bytes32 indexed _assetId, address indexed _owner);   
    
    /////// FUNCTIONS ///////////////////////////
    
    // Allows to register a new asset (assetId) where the owner is the msg.sender of the request
    function register(bytes32 assetId, uint256 price) public validAddress(msg.sender) returns (bool success);
}

```

---
<a name="asset_lifecycle"></a>
## Asset Lifecycle

This section describes the life cycle of the asset registry in Ocean protocol.

<a name="registering-a-new-asset"></a><a name="ASE.001"></a>
### Registering a new Asset 

![Registering a new Asset](images/ASE.001.png "ASE.001")


In the above diagram the Agent and the Orchestration capabilities are implemented in the AGENT scope.
The registering of a new Asset involves the following implementations:

#### Ocean Agent API

It is necessary to expose a RESTful HTTP interface using the following details:

```
Reference: ASE.001
Path: /api/v1/assets
HTTP Verb: POST
Caller: The Asset PUBLISHER
HTTP Output Status Codes: 
    HTTP 202 - Accepted
    HTTP 400 - Bad request
    HTTP 401 - Forbidden
    HTTP 422 - Asset already exists
```

<a name="asset-insert-params"></a>
##### Input Parameters 

| Parameter | Type | Description |
|:----------|:-----|:------------|
|assetId    |string|Id of the Asset that was generated by the Keeper VM|
|owner      |string|Owner address. This parameter MUST be validated. This information will not be give as part of the Payload, it will be retrieved from the Authorization HTTP header|
|marketplaceId    |string|Id of the Marketplace (optional). It indicates if the asset was published through a specific marketplace|
|metadata   |json|Attribute able to include any kind of additional information about the asset (optional)|
|hash       |string|metadata hash (optional) where agent and keeper VM will keep tracking the changing in metadata. It also could be generated automatically using the metadata|

Because all parameters are optional, an empty payload is allowed to create an Asset.
In the composition of the HTTP payload, only the assetId and marketplaceId will be in the root of the JSON document. The rest of the parameters (optional), will be included as part of the Metadata entity.

Example: 

```json
{	
    "assetId": "342094823423",
	"marketplaceId": "0xaabbccdd",
	"metadata": {
        "name": "transaction logs jan.2018",
        "mimeType": "text/csv",
        "attributes": [{
                "key": "description",
                "value": "company transaction logs"
            },
            {
                "key": "generatedDatetime",
                "value": "Jan/2018"
            }
        ],
        "parameters": [],
        "links": [{
            "name": "dataSample",
            "linkedAssetId": "0x12345678",
            "linkedType": "subset",
            "url": "http://example.com/samples/tx/2018/01.csv"
        }]
	}
}
```

Attributes might include extra information about the access, permissions, policies and service level agreement as shown below:

```json
{	
    //...
	"metadata": {
        //...   
        "attributes": [{
               "responseType": "Signed_URL",
                "resourceServerPlugin": "Azure",
                "serviceEndpoint": "https://provider123.com/cosume/?assetid=123344556",
                "permissions":{
                    "READ":true,
                    "WRITE": false,
                    "DELETE": false,
                    "UPDATE": false
                },
                "ServiceLevelAgreement":{
                    "type": "PDF",
                    "reference": "https://provider123.com/sla/123sla.pdf"
                }
                
             }]
        
        },
      //...
}
```

<a name="asset-model"></a>
##### Output 

The output of this request MUST add the following attributes generated by the system:

| Attribute         | Type      | Description |
|:------------------|:----------|:------------|
|creationDatetime   |datetime   |Allocated by the system when was created in the AGENT (universal datetime), time before consensus |
|updateDatetime     |datetime   |Allocated by the system when was updated the metadata in the AGENT (universal datetime), time before consensus |
|contentState       |string     |Internal state of the Asset (see below for a complete description) |


These are the possible values of the **contentState** attribute:

* PENDING - Asset just created
* **PUBLISHED - Asset available by at least one provider**
* UNPUBLISHED - An asset is not being provided
* DISABLED - Asset deleted


Example:

```json
{
    "assetId": "123456789abcde", 
  	"marketplaceId": "0xaabbccdd",
    "owner": "0x1234aa33bb",
    "creationDatetime": "2018-05-18T16:00:00Z",
    "updateDatetime": "2018-05-18T16:00:00Z",
    "contentState": "PENDING",
    "metadata" : {
        "name": "transaction logs jan.2018",
        "mimeType": "text/csv",
        "attributes": [ .. ],
        "parameters": [ .. ],
        "links": [ .. ]
	}
}
```

#### Registration flow

The AGENT node will be in charge of manage the Assets creation. The Ocean DB integration is optional, so the Metadata will be stored there only an Ocean DB interface implementation is provided.  
Assets MUST be persisted in the keeper VM and Ocean DB (if the configuration is provided), storing in the Decentralized DB only the essential information to run the Smart Contracts.
 
Ocean DB will store the complete metadata information. To coordinate the creation of the Assets in a consistent way in both data stores, the AGENT will implement an Orchestration component in charge of that.

The publisher will coordinate the creation of an Asset directly by writing initially in the keeper VM. 
It will emit an event <code>AssetRegistered</code> and return a **Transaction Receipt** (see more details about the [Transaction Receipt model](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethgettransactionreceipt)).
After of that the agent will persist the complete Asset metadata in Ocean DB.



#### Interaction with Ocean DB

If the **AGENT::Ocean DB** is integrated, it will persist the following information:

| Parameter | Type | Description |
|:----------|:-----|:------------|
|assetId    |string|Id of the Asset|
|owner    |string|Account address|
|marketplace    |string|Id of the Marketplace (optional)|
|state       |string     |Internal state of the Asset|
|**Metadata**:|     ||
|creationDatetime   |datetime   |Allocated by the system when was created in the AGENT (universal datetime), time before consensus |
|updateDatetime     |datetime   |Allocated by the system when was updated the metadata in the AGENT (universal datetime), time before consensus |
|name       |string|Asset name (optional)|
|mimeType       |string|The mime-type of the file (optional)|
|attributes |array |Array of key, value attributes (optional)|
|parameters |array | If it's a service or operation, specifies the K,V parameters (optional)|
|links |array |List of links to other assets (samples, previous versions, etc.) (optional)|
|providers       |object     |Providers giving access to the data. This is managed by the [**ASE.005**](#provider-asset) operation.|
|responseType|string|The expected response type. It might include signed url, ssh keys, or one time password|
|resourceServerPlugin|string|An internal attribute that is used by provider backend in order to resolve which provider plugin will be used to server to server the dataset|
|permissions|object|An object contains a key/value pairs for the assigned permissions for the asset or the resource|
|serviceLevelAgreement|object|This object contains two basic keys, a reference link for the service level agreement and the type of the file (XML, PDF, HTML, IPFS based hash)|  




<a name="retrieve-asset"></a><a name="ASE.002"></a>
### Retrieve metadata of an Asset 

![Retrieve metadata of an Asset](images/ASE.002.png "ASE.002")

The retrieval of an Asset involves the following implementations:

#### Ocean Agent API

It is necessary to expose a RESTful HTTP interface using the following details:

```
Reference: ASE.002
Path: /api/v1/assets/{assetId}
HTTP Verb: GET
Caller: Any User
Input: assetId
Output: Asset Schema
HTTP Output Status Codes: 
    HTTP 200 - OK
    HTTP 400 - Bad request
    HTTP 404 - Not Found
```

##### Input Parameters

| Parameter | Type | Description |
|:----------|:-----|:------------|
|assetId    |string|Id of the Asset|
|metadata hash|string|hash of asset metadata (optional)| 

Example: 

```http
GET http://localhost:8080/api/v1/assets/777227d45853a50eefd48dd4fec25d5b3fd2295e
```


<a name="update-asset"></a><a name="ASE.003"></a>
### Updating Asset Metadata 

![Updating Asset Metadata](images/ASE.003.png "ASE.003")

In the above diagram the Agent and the Orchestration capabilities are implemented in the AGENT. Asset involves the following parameters that could be updated:

#### Ocean Agent API

It is necessary to expose a RESTful HTTP interface using the following details:

```
Reference: ASE.003
Path: /api/v1/assets
HTTP Verb: PUT
Caller: The Asset PUBLISHER or MARKETPLACE on behalf
Input: Asset Schema
Output: Asset Schema
HTTP Output Status Codes: 
    HTTP 202 - Accepted
    HTTP 400 - Bad request
    HTTP 401 - Forbidden
```

##### Input Parameters

Input parameters accepted are the same are accepted in the [input parameters section](#asset-insert-params) of the Registering an Asset method.

Internal state attributes and the rest of the attributes can't be modified using this method.


##### Output

The expected output implements the Asset model described in the [output parameters section](#asset-model) of the Registering an Asset method.


#### Updating Asset flow

The AGENT node will be in charge of manage the Assets update by sending <code>Asset update</code> signal to 
ocean market contract in order to commit the new <code>metadata hash</code> and keep track the metadata version
 in smart contract, then make a call to the ocean DB in order to commit the final update. 


<a name="retire-asset"></a><a name="ASE.004"></a>
### Retiring an Asset 

![Retire an Asset](images/ASE.004.png "ASE.004")

In the above diagram the Agent and the Orchestrator capabilities are implemented in the AGENT scope.
The Asset MUST be retired from Ocean DB and the keeper VM.

This method implements a soft delete of an Asset. It means the Asset is updated setting the contentState attribute to `DISABLED`. The method will return a HTTP 202 status code and the Asset modified in the response body.

This method only can be integrated by the owner of the Asset or a Marketplace allowed to act on behalf.

#### Ocean Agent API

It is necessary to expose a RESTful HTTP interface using the following details:

```
Reference: ASE.004
Path: /api/v1/assets/{assetId}
HTTP Verb: DELETE
Caller: The Asset owner or Marketplace on behalf
Input: assetId
Output: Asset Schema
HTTP Output Status Codes: 
    HTTP 202 - Accepted
    HTTP 400 - Invalid params
    HTTP 404 - Not Found
```

##### Input Parameters

| Parameter | Type | Description |
|:----------|:-----|:------------|
|assetID    |string|The Asset ID given in the URL|


##### Output

The expected output implements the Asset model described in the [output parameters section](#asset-model) of the Registering an Asset method.

After execute this method, if everything worked okay, the **state** attribute should be **DISABLED**.


#### Orchestration Layer

The publisher will send a signal to the market contract in order to retire an asset. This asset will be no longer accessible throught blockchain.
 As a result, the agent will be notified by the emitted event <code>AssetRetired</code>, and make a call to delete this asset from Ocean DB.


### Assignee(s)
Primary assignee(s): @aaitor, @diminator


### Targeted Release

The implementation of the full Keeper functionality it's planned for the [Alpha release](https://github.com/oceanprotocol/ocean/milestone/4)


### Status
unstable


<a name="copyright-waiver"></a>
## Copyright Waiver  
To the extent possible under law, the person who associated CC0 with this work has waived all copyright and related or neighboring rights to this work.

