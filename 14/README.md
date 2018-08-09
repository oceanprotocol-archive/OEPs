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
      * [Token Curation Registry](#token-curation-registry)
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

* ASSETS are DATA objects describing RESOURCES under control of a PUBLISHER
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
As general rule, only the **INDISPENSABLE** information to run the Smart Contracts MUST be stored in te Decentralized VM.
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
* Only the very basic information about the Assets (ids and pricing) MUST be stored in the Decentralized VM too 
* AGENT MUST NOT store any information about the Assets or Actors during this process



The proposed solution is composed by the interaction of different elements:

* A high level HTTP API exposing the methods required to manage the Assets Registry (AGENT)
* A Keeper node registering the complete Assets metadata (KEEPER - Ocean DB). This is optional and depends of the user parameters.
* A Keeper node registering the Asset IDs (KEEPER - Decentralized VM)
* A backend orchestration layer (AGENT) in charge of coordinating the persistence of the Assets in both backends consistently (Ocean DB & Decentralized VM)

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
* The AGENT MUST orchestrate the Asset registering in the OCEAN DB and DECENTRALIZED VM
* The DECENTRALIZED VM MUST only store as less information as possible. Only the main IDs and pricing information
* The OCEAN DB MUST store the complete ASSET Metadata if it's configured/instantiated (this is OPTIONAL)
* The AGENT MUST validate the basic parameters sent by the PUBLISHER
* The AGENT MUST authenticate the PUBLISHER sending the request
* The AGENT MUST authorize the user via KEEPER
* The OCEAN DB MUST store the relation between the ASSET and the PROVIDER if it's configured/instantiated (this is OPTIONAL)


In the below diagram you can see an alternative interaction where the Marketplace is not involved: 

![high level interactions](images/arch-assets-interactions-nomkt.png)

The following sections will describe the end to end implementation using a top to bottom approach, 
starting from the API interface to the Keeper implementation, using the Ocean DB and the Decentralized VM.

In the following diagram you can see the nodes involved in this OEP:

![Nodes Involved](images/nodes-involved1.png "Nodes Involved")

In the following sections you can find the end to end implementation details of the complete OAR functionality.

### Smart Contracts

The KEEPER will store only the essential user information to allow the implementation of the Assets Registry. It means from the KEEPER side, the system MUST NOT store any kind of metadata.

Taking this into account, the skeleton of main implementation should provide the following structs and interfaces:

```solidity
contract AssetsRegistry {

    uint256 constant STATE_PENDING = 0; // Asset just created
    uint256 constant STATE_PUBLISHED = 1; // Asset used by at least one provider
    uint256 constant STATE_UNPUBLISHED = 2; // Asset is not being provided 
    uint256 constant STATE_DISABLED = 9; // Asset deleted

    struct Asset {
        address owner;
        address marketplace;
        uint256 state;
    }
    
    mapping(byte32 => Asset) assets;
    
    /////// EVENTS //////////////////////////////
    event AssetRegistered(bytes32 indexed _id, address indexed _mktId);    
    
    event AssetUpdated(bytes32 indexed _id, address indexed _mktId, uint256 indexed state);    
      
    event AssetAttributeChanged(bytes32 indexed _id, address indexed _mktId, bool indexed _isValid, bytes32 _name, bytes32 _value);    
    
    /////// FUNCTIONS ///////////////////////////
    
    // Allows to register a new asset (assetId) where the owner is the msg.sender of the request
    // An optional marketplaceId can be given, allowing to it to act in behalf of the user related with the asset
    function publish(bytes32 _id, address _mktId) external returns (bool success);
    
    // Remove the asset of the system (soft delete). 
    // Only the owner of the asset or marketplace associated can retire an asset
    function retire(bytes32 _id) external onlyOwnerOrMarketplace returns (bool success);
    
    // Returns the address of the owner of a specific asset
    function getOwner(bytes32 _id) external view returns (address owner);
    
    // Returns true or false saying if the address passed as parameter is the owner of the asset
    // or can act in behalf
    function isOwner(bytes32 _id, address _address) internal view returns (bool isOwner);
    
    // Returns true or false saying if the message.sender can update an asset    
    function canUpdate(bytes32 _id) internal view returns (bool success);
    
    // Update the internal state of an assent
    // Only the owner or marketplace in behalf can update the state
    function updateState(bytes32 _id, address _address, uint256 _newState) internal onlyOwnerOrMarketplace returns (bool success);
    
    // Returns true or false saying if the message.sender can retire an asset
    function canRetire(bytes32 _id) internal view returns (bool success);

    // Sugar on top of updateState method. Updates the state to DISABLED
    // Only the owner or marketplace in behalf can update the state  
    function retire(address _id, address _mktId) external onlyOwnerOrMarketplace returns (bool success);

    // Add an Attribute associated to an Asset
    // Attributes are stored as Events. It raises the AssetAttributeChanged event
    function setAttribute(bytes32 _id, bytes32 _key, bytes32 _value) external onlyOwnerOrMarketplace returns (bool success);

    // Revoke an Attribute associated to an Asset
    // Attributes are stored as Events. It raises the AssetAttributeChanged event
    function revokeAttribute(address _id, bytes32 _name, bytes32 _value) external onlyOwnerOrMarketplace returns (bool success);
    
}

```

Different states are:

* PENDING (state = 0) - Asset just created
* PUBLISHED (state = 1) - Asset used by at least one provider
* UNPUBLISHED (state = 2) - Asset is not being provided 
* DISABLED (state = 9) - Asset deleted

To save costs, the states are mapped to uint. Additional attributes required by the Actors TCR could be required.


---
<a name="asset_lifecycle"></a>
## Asset Lifecycle

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
|assetId    |string|Id of the Asset (optional). If not assetId is provided, the system will generate the id|
|owner      |string|Owner address. This parameter MUST be validated. This information will not be give as part of the Payload, it will be retrieved from the Authorization HTTP header|
|marketplaceId    |string|Id of the Marketplace (optional). It indicates if the asset was published through a specific marketplace|
|metadata   |json|Attribute able to include any kind of additional information about the asset (optional)|

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
* PUBLISHED - Asset used by at least one provider
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

#### Orchestration Layer

The AGENT node will be in charge of manage the Assets creation. The Ocean DB integration is optional, so the Metadata will be stored there only an Ocean DB interface implementation is provided.  
Assets MUST be persisted in the Decentralized VM and Ocean DB (if the configuration is provided), storing in the Decentralized DB only the essential information to run the Smart Contracts.
 
Ocean DB will store the complete metadata information. To coordinate the creation of the Assets in a consistent way in both data stores, the AGENT will implement an Orchestration component in charge of that.

The AGENT will coordinate the creation of an Asset writing initially in the Decentralized VM. It will return a **Transaction Receipt** (see more details about the [Transaction Receipt model](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethgettransactionreceipt)).
After of that the Orchestration layer will persist the complete Asset metadata in Ocean DB.

![Orchestration Layer behaviour](images/arch-orchestration-with-caching.png)


<a name="new-asset-vm"></a>
#### Interaction with the Decentralized VM

The **KEEPER::Decentralized VM** will persist the following information:

| Attribute | Type | Description |
|:----------|:-----|:------------|
|assetId    |byte32|Asset Id|
|owner      |address|Owner of the Asset|
|marketplace|address|Address of the Marketplace associated to the asset (optional)|
|state      |uint|TCR State of the Asset|

Using any of the existing web3 implementation library (web3.js, web3.py, web3.j, etc), it's possible to interact with the VM Smart Contracts.

For example using the java Smart Contract stubs generated by web3.j (Java), it's possible to implement a DTO wrapping the Smart Contract interactions. 

```java

AssetsRegistry registry= AssetsRegistry.load(
    contractAddress,
    vm.getWeb3(),
    txManager,
    GAS_PRICE,
    GAS_LIMIT
    ));

TransactionReceipt receipt= registry.publish(newAsset.assetId).send();

```


#### Interaction with Ocean DB

If the **KEEPER::Ocean DB** is integrated, it will persist the following information:

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



---

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

Example: 

```http
GET http://localhost:8080/api/v1/assets/777227d45853a50eefd48dd4fec25d5b3fd2295e
```

Before to query the Decentralized VM, it's necessary to check the length and format of the assetId. 
If the length and format doesn't fit the standard address definition, the system should return a **HTTP 400** Invalid params message.


#### Interaction with the Decentralized VM

The Asset information will be retrieved directly from the **KEEPER::Decentralized VM** Smart Contract interfaces. 
It's necessary to validate that the ```state != DISABLED```.
Disabled Assets MUST return a ```HTTP 404 Not Found``` status code.   

##### Output

The essential information is coming from the Decentralized VM. The expected output will provide the following attributes: 

| Attribute | Type | Description |
|:----------|:-----|:------------|
|assetId    |byte32|Asset Id|
|owner      |address|Owner of the Asset|
|marketplace|address|Marketplace associated to the asset (optional)|
|state      |uint|TCR State of the Asset|

If Ocean DB is enabled, the system will try to retrieve the complete Metadata using the interfaces and **assetId**. 
If the system is able to do it, the output will include in the metadata section all the extended information provided by the external system. 


---

<a name="update-asset"></a><a name="ASE.003"></a>
### Updating Asset Metadata 

![Updating Asset Metadata](images/ASE.003.png "ASE.003")

In the above diagram the Agent and the Orchestration capabilities are implemented in the AGENT scope.
The registering of a new Asset involves the following implementations:

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


#### Orchestration Layer

The AGENT node will be in charge of manage the Assets update. 

This method only provide the capabilities to update the following attributes in the the Decentralize VM:

* **marketplace**   

In addition to this, integrates the Decentralized VM to implement the access control mechanism.
The Access Control checks if the user requesting to Update the Asset, has enough privileges to do it. 
To do that, the ```canUpdate(assetId)``` method is called. This method should return a boolean value indicating if the Asset can be modified.  

If the Asset can be updated, and Ocean DB is enabled, the Orchestration layer will persist the complete Asset metadata in Ocean DB. 
Also the attribute **updateDatetime** will be updated with the KEEPER universal datetime. 


---

<a name="retire-asset"></a><a name="ASE.004"></a>
### Retiring an Asset 

![Retire an Asset](images/ASE.004.png "ASE.004")

In the above diagram the Agent and the Orchestrator capabilities are implemented in the AGENT scope.
The Asset MUST be retired from Ocean DB and the Decentralized VM.

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


Example: 

```http
DELETE http://localhost:8080/api/v1/assets/777227d45853a50eefd48dd4fec25d5b3fd2295e
``` 

##### Output

The expected output implements the Asset model described in the [output parameters section](#asset-model) of the Registering an Asset method.

After execute this method, if everything worked okay, the **state** attribute should be **DISABLED**.


#### Orchestration Layer

The AGENT node will be in charge of manage the Assets retirement. 
If Ocean DB is enabled, Assets MUST be updated in Ocean DB and retired from the Decentralized VM.

Before to proceed to any data modification, it's necessary to validate if the address requesting to retire an Asset has privileges to do it. 
The Access Control component is in charge of this validation (KEEPER). To do that, the ```canRetire(assetId)``` method is called. This method should return a boolean value indicating if the Asset can be retired by the user calling that method.  

If the Asset can be retired, the Smart Contract will execute the **retire** method.

After to do that, the Orchestration layer will update the following information in Ocean DB:
* The attribute **state** will be updated with the value **DISABLED**.
* The attribute **updateDatetime** will be updated with the KEEPER universal datetime. 

The AGENT will coordinate the retirement of an Asset interacting initially with the Decentralized VM. It will return a **Transaction Receipt** (see more details about the [Transaction Receipt model](https://github.com/ethereum/wiki/wiki/JavaScript-API#web3ethgettransactionreceipt)).
After of that the Orchestration layer will update the above attributes in Ocean DB if it's enabled.


---

<a name="asset-tcr"></a>
### Token Curation Registry

Token curation registry (TCR) is used to maintain a list of high quality assets through challenge-voting process:

* Voting process can be initiated by:
	- New Asset applies to be listed in the marketplace
	- Existing Asset is challenged by any user
	- All participants including applicant, voter and challenger need to deposit tokens for challenge or voting
	- Deposits of minority in voting result will be distributed to majority party
* Each participant can vote for or against the asset according to their opinion.
* After the voting result is revealed, the token deposit will be distributed among winning parties.
* Depends on the voting result, the asset will be accepted to be listed or removed from the marketplace. 

#### TCR Smart Contract

The TCR Smart Contract can be implemented as a stand-alone module which interacts with the Assets Registry through Interface Functions. 

Let us introduce the data struct and functions of TCR smart contract first, and then discuss the interaction to Assets Registry smart contract.

The TCR smart contract SHOULD provide the structs including `listing` and `challenge`:  

```solidity
    // Maps assetId to associated listing data
    mapping(string => Listing) public listings;
    
    // Maps challengeIDs to associated challenge data
    mapping(uint => Challenge) public challenges;
    
	// Listing data struct
	struct Listing {
		uint		version;
		uint		applicationExpiry;
		bool		whitelisted;
		uint		challengeID;
		string		description;
	}
	
	
	// Challenge data struct
	struct Challenge {
		uint		rewardPool;
		address		challenger;
		uint		stake;
		uint		totalTokens;
		bool		resolved;
		mapping(address => bool) tokenClasims; 
	}  
```

The **Listing** object includes the following attributes:

| Parameter | Type | Description |
|:----------|:-----|:------------|
|version | uint | the version of TCR contract |
|applicationExpiry  |uint| Expiration date of application stage|
|whitelisted  |bool   | Indicates registry status|
|challengeID       |uint| voting ID in challenge stage|
|description|string|Description about the TCR (optional)|

The **Challenge** object includes the following attributes:

| Parameter | Type | Description |
|:----------|:-----|:------------|
|rewardPool  |uint| pool of tokens to be distributed to winning voters|
|challenger | address| owner of challenge|
| resolved | bool | indicates challenge is resolved |
|stake | uint | number of tokens at stake |
|totalTokens | uint | number of tokens used in voting by the winning side |
|tokenClasims | mapping(address=>bool) | indicates whether a voter has claimed reward |

#### Interaction with the Asset Registry

The Asset Registry Smart Contract SHOULD provide following methods to interact with TCR in two scenarios:

* *Case 1: registering new dataset initiates the voting process when TCR is enabled:*
	* The function `applyListAsset` creates a challenge of `_assetId`; 
	* The challenge triggers the voting process;
	* Depends on the result, the function returns "true" if most voters agree to list the new dataset and new dataset can be listed in the marketplace;
	* It returns "false" if not and new dataset is disabled. 

	```solidity
		function applyListAsset(
	        bytes32 _assetId, 
	        address _providerId) 
	    external returns (bool success);
	
	```

* *Case 2: User challenges the existing asset:*
	* User creates a challenge of "_assetId" and triggers the voting process with TCR Smart Contract;
	* The TCR Smart Contract SHOULD call `retire` methods in Asset Registry if voting result is to remove the asset from the marketplace;
	* As such Asset Registry smart contract removes the asset.

	```solidity
	function retire(bytes32 _assetId) external returns (bool success);
	```
	

---

#### Ocean Agent API

### Assignee(s)
Primary assignee(s): @aaitor, @diminator


### Targeted Release

The implementation of the full Keeper functionality it's planned for the [Alpha release](https://github.com/oceanprotocol/ocean/milestone/4)


### Status
unstable


<a name="copyright-waiver"></a>
## Copyright Waiver  
To the extent possible under law, the person who associated CC0 with this work has waived all copyright and related or neighboring rights to this work.

