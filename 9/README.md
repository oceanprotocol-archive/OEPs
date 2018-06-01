```
shortname: 9/TX
name: Ocean Order Transactions
type: Standard
status: Raw
editor: Aitor Argomaniz <aitor@oceanprotocol.com>

```

<!--ts-->

Table of Contents
=================

   * [Table of Contents](#table-of-contents)
   * [Ocean Order Transactions](#ocean-order-transactions)
      * [Change Process](#change-process)
      * [Language](#language)
      * [Motivation](#motivation)
      * [Specification](#specification)
      * [Proposed Solution](#proposed-solution)
         * [Smart Contracts](#smart-contracts)
         * [Create Contract](#create-contract)
         * [Retrieve Contract](#retrieve-contract)
         * [Sign a Contract](#sign-a-contract)
         * [Authorize Consumption](#authorize-consumption)
         * [Get Consumption Information](#get-consumption-information)
         * [Revoke Access](#revoke-access)
      * [Additional info](#additional-info)

      
<!--te-->

<a name="ocean-order-transactions"></a>
# Ocean Order Transactions

The Ocean Order Transactions (**TX**) is a specification for Ocean Protocol to manage the interaction between users trying to negotiate the Asset access or consumption through the network.

This OEP does not focus on asset attributes, whitelisting or curation. It's purely the mechanics of registering the order transactions allowing to the Ocean users to trade using the protocol.

This specification is based on [Ocean Protocol technical whitepaper](https://github.com/oceanprotocol/whitepaper), [3/ARCH](../3/README.md), [4/KEEPER](../4/README.md) and [5/AGENT](../5/README.md).

This specification is called **TX** henceforth.

<a name="change-process"></a>
## Change Process
This document is governed by the [2/COSS](../2/README.md) (COSS).

<a name="language"></a>
## Language
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.

<a name="motivation"></a>
## Motivation

Ocean network aims to power marketplaces for relevant AI-related data services.
Different actors and stakeholders are necessary to interact between them using the Ocean Protocol defined.
One of the essential points of Ocean is to provide the mechanisms allowing to untrusted users to trade using the system.  

The main motivation of TX is to define a simple and clear protocol allowing those mechanics.

Some considerations:

* The negotiation protocol SHOULD be as simple as possible
* The protocol SHOULD be flexible enough to support different scenarios
* The information to store on-chain MUST be only the essential information to run the Smart Contracts
* Marketplaces are actors facilitating the discovery/negotiation but are not indispensable 
* Protocol MUST support contract definition and consumption without Marketplaces  

<a name="specification"></a>
## Specification 

Main requirements are:

* TX MUST provide the Smart Contracts logic allowing to non-trusted parties to negotiate the ASSETS sharing
* Main TX business logic will be implemented in the Smart Contract layer running in the **KEEPER::DEC-VM**
* The AGENT will interface with the KEEPER::DEC-VM to compose the valid transactions needed by the Smart Contracts layer
* The AGENT will integrate the KEEPER::DEC-VM to implement the Access Control capabilities
* The information sent to the KEEPER::DEC-VM by the AGENT MUST be only the essential
* The AGENT will integrate with OCEAN DB if it's provided, allowing to send to an external Data Store the copy of the transactions managed
* The AGENT and OCEAN DB integration is totally optional, the only source of truth is the data stored on-chain in the KEEPER::DEC-VM
* The AGENT will provide a higher level API allowing to third parties to integrate with TX in an easier way
* Contract negotiations MUST occur off-chain, the protocol won't provide methods for ASK/MATCHING
* Main parties involved in a Contract negotiation are PUBLISHERS, PROVIDERS, CONSUMERS and MARKETPLACES
* MARKETPLACES are facilitators but not indispensable for a Contract settlement
* A Contract can be prepared or defined by any of the parties involved in the Contract. 
* The essential information about the contract MUST be stored on-chain
* All the parties of the contract MUST sign the contract before to move to any consumption phase
* If a contract is not signed after a time, the contract SHOULD be Cancelled
* If a contract is signed, the PROVIDER is the one Authoring the Consumption
* When a contract is authorized, the CONSUMER can request the access to the ASSETs or SERVICES listed in the Contract
* When the PROVIDER facilitates all the Proofs of Service requested, the contract is moved to a settlement state

The following restrictions apply during the design/implementation of this OEP:

* The Assets registered in the system MUST be associated to the Actors registering the Assets
* The Actors associated to the Contracts (PUBLISHER, PROVIDER, CONSUMER, MARKETPLACE) MUST have a valid Account Id in the system
* The information or Metadata about the Contracts will be stored in Ocean DB if the user plugs a valid Ocean DB implementation
* Only the very basic information about the Contracts (ids, proofs, etc) MUST be stored in the KEEPER::DEC-VM
* AGENT MUST NOT store any information about the Contracts, Assets or Actors during this process


![stack view](images/contracts-states.png)

A Contract could implement a state machine with the following states:

* **DRAFT** - Contract is created on-chain by any of the parties involved. Contract can be created as a result of a negotiation (Bid/Match) or automatically (ie. free scenarios). During the Creation, one or many signatures related with the parties involved can be provided.
* **SIGNED** -  All the parties involved in the Contract have provided their signature.
* **AUTHORIZED** - The provider authorize the consumption of the Asset giving (encrypted) the Consumption information (url, user, password, etc.)
* **SETTLED** - After being challenged and all the Proofs of Services requested provided, the contract is Settled. 
* **CANCELLED** - The Contract is cancelled by any reason. Signatures or authorization not provided, etc. 

The system MUST be designed and developed with PRIVACY and ANONYMITY as core principles. 

The **Contracts** information should be managed using an API. This API should exposes the following capabilities:

* Create a Contract
* Retrieve a Contract
* Sign a Contract
* Authorize a Contract
* Request access to the contents included in a contract
* Settle a Contract
* Revoke Contract Authorization


<a name="proposed-solution"></a>
## Proposed Solution 

The proposed solution is composed by the interaction of different elements:

* A high level RESTful API exposing the methods required to manage the Contracts (AGENT)
* A Keeper node registering the Contracts on-chain (KEEPER::DEC-VM)
* Optionally, an external database pluged using the Ocean DB interfaces 

![Nodes Involved](images/nodes-involved.png)

We can show the interaction between layers and components using a stack view:

![stack view](images/arch-actors-stack.png)

The following sections will describe the end to end implementation using a top to bottom approach, starting from the API interface to the Keeper implementation.

![stack view](images/main-interactions.png)

The above diagram shows the high level interactions between the components involved:

* A CONSUMER, using a Marketplace frontend application find an Asset and click on "buy button"
* The MARKETPLACE using his local AGENT or the CONSUMER, register the Contract in the KEEPER::DEC-VM ([CON.001](#CON.001)). In this process the Marketplace could provide the public keys of some of the parties involved (Marketplace, Consumer, Provider, Publisher)
  - In scenarios where the conditions are agreed by the Consumer, won't be necessary to request the approval/sign of the Publisher and Provider. It would reduce the time for contract setup, allowing quick consumption from the Consumer side
  - For example optionally, because the Asset was registered through the Marketplace, the Marketplace send a notification to the Publisher requesting his/her signature on the contract. If the user is a registered user in the Marketplace, this notification can be sent by email, sms or pidgeon, doesn’t matter.
* The PUBLISHER receives the sign notification and provides his/her signature through the Marketplace frontend application
* The MARKETPLACE using the AGENT provide the PUBLISHER signature and store on-chain ([CON.003](#CON.003))
* The PROVIDER AGENT is subscribed to the KEEPER::DEC-VM transaction log. So the Provider is aware of any change in any Contract related to him. 
* When the Contract has been signed by all the parties, the PROVIDER AGENT send an Authorization Consumption request ([CON.004](#CON.004)). The PROVIDER gives the information necessary to consume the ASSET (it includes where is the asset and how to get access). This information is encrypted using asymmetric cryptography. So only the CONSUMER using his/her private key should be able to get decrypt this data.  This is stored on-chain. 
* The MARKETPLACE AGENT, subscribed to the transaction log, after to get notice the contract has been signed and access granted, notify the CONSUMER saying the ASSET is ready to be consumed.
* The CONSUMER request access to download the asset [CON.005](#CON.005) using the information provided on-chain by the PROVIDER.
* The PROVIDER AGENT the Access Control layer validate the access grants of the CONSUMER to the ASSET related to the Contract. This is validated using the KEEPER::DEC-VM. On-Chain Access Control.
* If everything is okay, the PROVIDER returns the content of the ASSET in the same request.

The workflow described is one of the available options. Because the protocol is flexible, **alternative workflows can be implemented**: 

* During the creation of the Contract, the Marketplace could provide the Publisher signature during the Contract creation. This depends of the Marketplace conditions provided to their users.
* In a Free scenario, all the signature process could be automated without any kind of interaction
* In a scenario where the conditions requested by Producer and Publisher are agreed by the Consumer, the Contract won't require Publisher and Provider signatures, moving to **SIGNED** state automatically after the Consumer (and Marketplace if it's involved) signs the contract
* The transaction log of the KEEPER::DEC-VM is open, so any actor could be subscribed to them to be aware of the Contract changes for example. It could be implemented without any Agent, using directly the web3 libraries.
* The request access/download of an Asset could happen also having the Marketplace as intermediary
* The Marketplace is a facilitator, but all the contract negotiation and definition could happen without any interaction of the Marketplace. Publishers, Providers and Consumers could complete a full transaction with a Marketplace. 
* Different roles can be provided by the same actor, it would allow alternative scenarios also supported:
  - Publisher acting in addition as Providers giving access to the Assets
  - Marketplaces acting as Providers
  - Marketplaces acting on behalf of Providers or Publishers 


In the following sections you can find the end to end implementation details of the complete TX functionality.



<a name="smart-contracts"></a>
### Smart Contracts 

The **KEEPER::DEC-VM** will store the essential user information to allow the implementation of the TX OEP.
It means the system MUST NOT store any personal information, enabling PRIVACY and ANONYMITY.

Taking this into account, the skeleton of main implementation should provide the following structs and interfaces:

```solidity

contract ContractsRegistry {

        
    uint256 constant STATE_DRAFT= 0;
    uint256 constant STATE_SIGNED= 1;
    uint256 constant STATE_AUTHORIZED= 2;
    uint256 constant STATE_SETTLED= 3;
    uint256 constant STATE_CANCELED= 9;
        
    struct Contract {
       
        bytes32 contractId;
        
        // Parties involved 
        address publisherId;
        address consumerId;
        address providerId;        
        address marketplaceId= 0x0; // Optional party
        
        bytes32 assetId;
        
        // 0= DRAFT, 1=SIGNED, 2=AUTHORIZED, 3=SETTLED, 9=CANCELED
        uint state;
        ContractCondition conditions;        
    }
    
    struct ContractConditions {
        uint price;
        uint availability;
    }
    
    mapping(bytes20 => Contract) contracts;
    
    
    /////// EVENTS //////////////////////////////
    event ContractRegistered(address indexed _pubId, address indexed _proId, address indexed _conId, address indexed _mktId, bytes32 indexed _assetId, bytes32 _contractId);
    
    event ContractSigned(bytes32 indexed _contractId, address _signer);
    
    event ContractUpdated(bytes32 indexed _contractId, unit256 indexed _state);
     
    
    /////// FUNCTIONS ///////////////////////////
    function register(address _pubId, address _proId, address _conId, address _mktId, bytes32 _assetId, uint price, uint availability) 
                        external returns (bytes32 contractId);
    
    function getState(bytes32 _contractId) public view returns (uint state);
    
    function getContract(bytes32 _contractId) public view 
                returns (address _pubId, address _proId, address _conId, address _mktId, bytes32 _assetId, uint _price, uint _availability);

    // Given an array of ids of actors signing the contract
    function signContract(bytes32 _contractId) external returns (bool success);

    function authorize(bytes32 _contractId) external returns (bool success);
    
    function provideAccess(bytes32 _contractId, bytes32 _url, bytes32 _user, bytes32 _passwd, bytes32 _token) external returns (bool success); 

    function getConsumptionInfo(bytes32 _contractId) external returns (bytes32 _url, bytes32 _user, bytes32 _passwd, bytes32 _token);

    function settle(bytes32 _contractId) external returns (bool success);
    
    function revoke(bytes32 _contractId) external returns (bool success);

}
```

Different states are:

* **DRAFT** - `state= 0`
* **SIGNED** -  `state= 1`
* **AUTHORIZED** - `state= 2`
* **SETTLED** - `state= 3` 
* **CANCELLED** - `state= 9` 

To save costs, the states are mapped to uint. 


---
<a name="create-contract"></a><a name="CON.001"></a>
### Create Contract

![Registering a new Contract](images/CON.001.png "CON.001")

In the [above diagram](diagrams/CON.001.md) the Agent is in charge of interacting with the KEEPER::DEC-VM to implement the Access Control validations and Contract definition on-chain.
If Ocean DB is enabled, the AGENT will send the Contract information to Ocean DB.
The registering of a new Contract involves the following implementations:

#### Ocean Agent API

It is necessary to expose a RESTful HTTP interface using the following details:

```
Reference: CON.001
Path: /api/v1/contracts
HTTP Verb: POST
Caller: Any party involved in the Contract (CONSUMER, PRODUCER, PUBLISHER, MARKETPLACE)
HTTP Output Status Codes: 
    HTTP 202 - Accepted
    HTTP 400 - Bad request
    HTTP 401 - Forbidden  
```

##### Input Parameters

| Parameter | Type  | Description |
|:----------|:------|:------------|
|assetId    |string |Id of the Asset related with the contract|
|publisherId|string |Id of the publisher|
|consumerId |string |Id of the consumer|
|providerId |string |Id of the provider (optional). A contract can be created as DRAFT without Provider|
|marketplaceId|string|Id of the marketplace (optional). A contract can executed without a Marketplace participation|
|price      |int    |Price defined in the contract in token drops. 0 if free.|
|availability|int   |Period of time the consumption is available (optional). 0 if there not any limitation|
|metadata   |Json   |Free Json object of information to be persisted in Ocean DB if enabled (optional)|

If **OCEAN DB** is enabled, the content of the Metadata attribute will be pass as parameter to the Ocean DB implementation to be stored in an external system.  

Example: 

```json
{	
	"assetId": "1234abcde",
	"publisherId": "0x999888777",
	"consumerId": "0xaabbcc112",
	"providerId": "0xaabbcc112",
	"price": 10,
	"metadata": {
        "topic": "xxx",
        "attributes": [{
            "key": "interests",
            "value": "Looking Ahead"
        }]
	}
}
```

<a name="contract-output"></a>
##### Output

The output of this request MUST add the following attributes generated by the system:

| Parameter | Type | Description |
|:----------|:-----|:------------|
|contractId|string|Id of the Contract created|
|assetId|string|Id of the Asset related with the contract|
|publisherId|string|Id of the publisher|
|consumerId |string|Id of the consumer|
|providerId |string|Id of the provider (optional)|
|marketplaceId |string|Id of the marketplace (optional)|
|price      |int    |Price defined in the contract in token drops. 0 if free.|
|availability|int   |Period of time the consumption is available (optional). 0 if there not any limitation|
|signedByPublisher      |bool    |Was the contract signed by the Publisher?|
|signedByProvider      |bool    |Was the contract signed by the Publisher?|
|signedByConsumer      |bool    |Was the contract signed by the Publisher?|
|signedByMarketplace      |bool    |Was the contract signed by the Publisher?|
|creationDatetime|datetime|Allocated by the system when was created in the AGENT (universal datetime), time before consensus|
|updateDatetime|datetime|Allocated by the system when was updated the metadata in the AGENT (universal datetime), time before consensus
|state|string|Internal state of the Contract: DRAFT, SIGNED, AUTHORIZED, SETTLED, CANCELLED|

Example: 

```json
{	
    "contractId": "87654321",
	"assetId": "1234abcde",
	"publisherId": "0x999888777",
	"consumerId": "0xaabbcc112",
	"providerId": "0xaabbcc112",
	"price": 10,
	"signedByPublisher": true,
	"signedByProvider": true,
	"signedByConsumer": false,
	"signedByMarketplace": false,
	"state": "DRAFT",
	"creationDatetime": "2018-05-18T16:00:00Z",
    "updateDatetime": "2018-05-18T16:00:00Z",
	"metadata": {
        "topic": "xxx",
        "attributes": [{
            "key": "interests",
            "value": "Looking Ahead"
        }]
	}
}
```

#### Orchestration Layer

The AGENT node will be in charge of manage the Contracts creation. 
The Ocean DB integration is optional, so the Metadata will be stored there only if Ocean DB interface implementation is provided.
Contract information MUST be persisted in the KEEPER::DEC-VM and Ocean DB (if the configuration is provided), storing in the Decentralized DB only the essential information to run the Smart Contracts.

Ocean DB will store the complete metadata information. To coordinate the creation of the Contracts in a consistent way in both data stores, the AGENT will implement an Orchestration component in charge of that.

The AGENT will coordinate the creation of an Contracts writing initially in the KEEPER::DEC-VM. It will return a Transaction Receipt (see more details about the Transaction Receipt model). 
After of that the Orchestration layer will persist the complete Contracts metadata in Ocean DB.



#### Interaction with the Keeper


The **KEEPER::DEC-VM** will persist the following information:

| Attribute | Type  | Description |
|:----------|:------|:------------|
|contractId |address|Id of the contract created|
|publisherId|address|Owner of the Asset|
|providerId |address|Owner of the Asset|
|consumerId |address|Owner of the Asset|
|marketplaceId|address|Owner of the Asset|
|assetId    |bytes32|Id of the Asset|
|state      |uint   |State of the Contract (0= DRAFT, 1=SIGNED, 2=AUTHORIZED, 3=SETTLED, 9=CANCELED)|
|price      |uint   |Price offered by the Consumer|
|availability|uint   |Time period the asset is available|
|signedByPublisher      |bool    |Was the contract signed by the Publisher?|
|signedByProvider      |bool    |Was the contract signed by the Publisher?|
|signedByConsumer      |bool    |Was the contract signed by the Publisher?|
|signedByMarketplace      |bool    |Was the contract signed by the Publisher?|

During the execution of the `register` function, the following tasks MUST be implemented:

* Check the authorization of the msg.sender
* Check the asset availability
* Check if the asset has assigned a provider
* Check if the conditions proposed (price & availability) meet the provider and publisher requirements
  - If conditions are granted, state is **SIGNED**
  - Else, state is **DRAFT**
* Register the contract 

Using any of the existing web3 implementation library (web3.js, web3.py, web3.j, etc), it's possible to interact with the VM Smart Contracts.



#### Interaction with Ocean DB

The integration with OCEAN DB is optional, so only will works if an implementation backend is provided.

If it's enabled, the Ocean DB layer will interact with the backend to store the metadata information about the contracts. So a part of the information defined in the Output section, the AGENT will forward the **metadata** json document to the Ocean DB implementation. 


---

<a name="retrieve-contract"></a><a name="CON.002"></a>
### Retrieve Contract

![Retrieve Contract information](images/CON.002.png "CON.002")

In the [above diagram](diagrams/CON.002.md), the retrieval of the Contract information state is related with the AGENT and the KEEPER::DEC-VM. No information is read from Ocean DB, unless an Ocean DB implementation is provided. This functionality involves the following implementations:

#### Ocean Agent API

It is necessary to expose a RESTful HTTP interface using the following details:

```
Reference: CON.002
Path:  /api/v1/contracts/{contractId}
HTTP Verb: GET
Caller: Any
HTTP Output Status Codes: 
    HTTP 200 - OK
    HTTP 400 - Invalid params
    HTTP 404 - Not Found
```


##### Input Parameters

| Parameter | Type | Description |
|:----------|:-----|:------------|
|contractId |string|Contract ID to retrieve|

Example: 

```http
GET http://localhost:8080/api/v1/contracts/48dd4fec25d5b3fd2295e
```

##### Output

A complete description of the Contract output can be found in the [previous section](#contract-output).

#### Interaction with the Keeper

Before to query the KEEPER::DEC-VM, it's necessary to check the length and format of the contractId. If the length and format doesn't fit the standard address definition, the system should return a **HTTP 400** Invalid params message.

The KEEPER::DEC-VM stores the state about the Contract, being the KEEPER::DEC-VM the source of truth about Contracts information. 
The AGENT will use the `AssetsRegistry.getContract` method to check if the contract already exists and return the Contract attributes.
If no contract is found, the system must return a **HTTP 404** Not Found message.

If Ocean DB is enabled, using the contractId as key, the system will retrieve the information about the Contract. So having the Contract information retrieved from the KEEPER::DEC-VM and Ocean DB, the AGENT should compose the output. 
Taking into account the information given by the KEEPER::DEC-VM should prevail. 




---
<a name="sign-contract"></a><a name="CON.003"></a>
### Sign a Contract


![Sign a Contract](images/CON.003.png "CON.003")

The objective of this function is to allow to any of the parties involved in a Contract to Sign the contract conditions. When all the parties have already signed the contract, this should be moved to the **SIGNED** state.

In the [above diagram](diagrams/CON.003.md) the AGENT is in charge of interacting with the KEEPER::DEC-VM to implement the Access Control validations and updating the contract with the provided signature on-chain.
If Ocean DB is enabled, the AGENT will send the update about the Contract information to Ocean DB.
This function involves the following implementations:

#### Ocean Agent API

It is necessary to expose a RESTful HTTP interface using the following details:

```
Reference: CON.003
Path: /api/v1/contracts/{contractId}/signature
HTTP Verb: PUT
Caller: Any party involved in the Contract (CONSUMER, PRODUCER, PUBLISHER, MARKETPLACE)
HTTP Output Status Codes: 
    HTTP 202 - Accepted
    HTTP 400 - Bad request
    HTTP 401 - Forbidden  
```

##### Input Parameters

| Parameter | Type  | Description |
|:----------|:------|:------------|
|contractId |string |Id of the contract (URL)|
|actorId    |string |Id of user signing the request, comes from the **HTTP Authorization header**|

If **OCEAN DB** is enabled, the content of the Metadata attribute will be pass as parameter to the Ocean DB implementation to be stored in an external system.  


<a name="contract-output"></a>
##### Output

A complete description of the Contract output can be found in the [previous section](#contract-output).

#### Orchestration Layer

The AGENT node will be in charge of manage the Contracts creation. 
The Ocean DB integration is optional, so the Metadata will be stored there only if Ocean DB interface implementation is provided.

The AGENT interact with the `AssetsRegistry::signContract`, allowing to the user sending the request `msg.sender` to sign the contract given as parameter.

#### Interaction with the Keeper

During the execution of the `register` function, the following tasks MUST be implemented:

* Check the authorization of the msg.sender
* Check the contract availability
* Update the signature state associated with the user public key
* Check if all the users provided the signature. if all the users signed the contract, the status should change to **SIGNED**


#### Interaction with Ocean DB

The integration with OCEAN DB is optional, so only will works if an implementation backend is provided.

If it's enabled, the Ocean DB layer will interact with the backend to store the metadata information about the contracts. 
So a part of the information defined in the Output section, the AGENT will forward the information related with the signature action and the change of state. 



---
<a name="authorize-consumption"></a><a name="CON.004"></a>
### Authorize Consumption


![Sign a Contract](images/CON.004.png "CON.004")

The objective of this function is to allow the Publisher to authorize the Asset consumption by the Consumer. This typically happens when all the users have signed the contract. 
This method should moved the Contract state to **AUTHORIZED**.

When the Publisher authorize the consumption, must provide all the information necessary to consume the Asset related with the Contract. This information MUST be encrypted, allowing only to the Consumer specified in the Contract to decrypt this information using his/her public & private keys. 

In the [above diagram](diagrams/CON.004.md) the AGENT is in charge of interacting with the KEEPER::DEC-VM to implement the Access Control validations and updating the contract state on-chain.
If Ocean DB is enabled, the AGENT will send the update about the Contract information to Ocean DB.
This function involves the following implementations:

#### Ocean Agent API

It is necessary to expose a RESTful HTTP interface using the following details:

```
Reference: CON.004
Path: /api/v1/contracts/authorize
HTTP Verb: PUT
Caller: PUBLISHER
HTTP Output Status Codes: 
    HTTP 202 - Accepted
    HTTP 400 - Bad request
    HTTP 401 - Forbidden  
```

##### Input Parameters

| Parameter | Type  | Description |
|:----------|:------|:------------|
|contractId |string |Id of the contract|
|actorId    |string |Id of user signing the request, comes from the **Authorization** header|
|**Consumption Information** |||
|url |string |Url used to access the asset (http://, tcp://)|
|token|string |Access token to the asset (optional)|
|user|string |User name (optional)|
|password|string |Password (optional)|

Example: 

```json
{	
	"contractId": "1234abcde342",
	"consumption": {
	  "url": "http://provider.com/assets/?assetId=xxxx",
	  "user": "access-user",
	  "password": "$-Df87Yj"
	}
}
```

<a name="contract-output"></a>
##### Output

A complete description of the Contract output can be found in the [previous section](#contract-output).

#### Orchestration Layer

The AGENT node will be in charge of manage the Contracts creation. 
If **OCEAN DB** is enabled, the content of the Metadata attribute will be pass as parameter to the Ocean DB implementation to be stored in an external system. 
The Consumption information will be stored on-chain and **WON'T** be send to Ocean DB. 

The AGENT interact with the `AssetsRegistry::provideAccess` method, allowing to the Consumer to get access to the Asset specified in the Contract. 
The PROVIDER should encrypt the Consumption information using the Consumer public key, allowing only to him/her to decrypt the Consumption information. 

#### Interaction with the Keeper

During the execution of the `provideAccess` function, the following tasks MUST be implemented:

* Check the contract availability
* Check the Contract is in **SIGNED** state
* Check the user trying to give access is the Provider
* Store the encrypted consumption information
* Change the state to **AUTHORIZED**


#### Interaction with Ocean DB

The integration with OCEAN DB is optional, so only will works if an implementation backend is provided.

If it's enabled, the Ocean DB layer will interact with the backend to store the metadata information about the contracts. 
The Consumption information will be stored on-chain and **WON'T** be send to Ocean DB. 
 
In this case the only information updated in Ocean DB is the **Contract state**.



---
<a name="get-consumption-information"></a><a name="CON.005"></a>
### Get Consumption Information

![Get Consumption Information](images/CON.005.png "CON.005")

In the [above diagram](diagrams/CON.005.md), the retrieval of the Contract information is related with the AGENT and the KEEPER::DEC-VM. 
No information is read from Ocean DB. This functionality involves the following implementations:

#### Ocean Agent API

It is necessary to expose a RESTful HTTP interface using the following details:

```
Reference: CON.005
Path:  /api/v1/contracts/{contractId}/consumption
HTTP Verb: GET
Caller: Any
HTTP Output Status Codes: 
    HTTP 200 - OK
    HTTP 400 - Invalid params
    HTTP 401 - Forbidden
    HTTP 404 - Not Found
```


##### Input Parameters

| Parameter | Type | Description |
|:----------|:-----|:------------|
|contractId |string|Contract ID to retrieve|
|actorId    |string |Id of user signing the request, comes from the **Authorization** header|

Example: 

```http
GET http://localhost:8080/api/v1/contracts/48dd4fec25d5b3fd2295e/consumption
```

##### Output


| Parameter | Type  | Description |
|:----------|:------|:------------|
|contractId |string |Id of the contract|
|**Consumption Information** |||
|url |string |Url used to access the asset (http://, tcp://)|
|token|string |Access token to the asset (optional)|
|user|string |User name (optional)|
|password|string |Password (optional)|

Example (data already de-crypted): 

```json
{	
	"contractId": "1234abcde342",
	"consumption": {
	  "url": "http://provider.com/assets/?assetId=xxxx",
	  "user": "access-user",
	  "password": "$-Df87Yj"
	}
}
```

The information returned to the user is encrypted by the Provider. Only the Consumer, using his/her public and private keys can de-crypt the Consumption Information. 
 

#### Interaction with the Keeper

Before to query the KEEPER::DEC-VM, it's necessary to check the length and format of the contractId. If the length and format doesn't fit the standard address definition, the system should return a **HTTP 400** Invalid params message.

The KEEPER::DEC-VM stores the state about the Contract, being the KEEPER::DEC-VM the source of truth about Contracts information. 
The AGENT will execute the `AssetsRegistry.getConsumptionInfo` function to try to return the encrypted Consumption Information to the Consumer. 
If no contract is found, the system must return a **HTTP 404** Not Found message.
If the user is not authorized to get the Consumption information, the system must return a **HTTP 401** Forbidden message.






---
<a name="revoke-access"></a><a name="CON.006"></a>
### Revoke Access



![Revoke Access](images/CON.006.png "CON.006")

The objective of this function is to revoke the signature to a previously signed contract. 
If any of the parties involved in the contract revoke access, the contract should be moved to the **CANCELLED** state.

In the [above diagram](diagrams/CON.006.md) the AGENT is in charge of interacting with the KEEPER::DEC-VM to implement the Access Control validations and updating the contract state on-chain.
If Ocean DB is enabled, the AGENT will send the update about the Contract information to Ocean DB.
This function involves the following implementations:

#### Ocean Agent API

It is necessary to expose a RESTful HTTP interface using the following details:

```
Reference: CON.005
Path: /api/v1/contracts/{contractId}/signature
HTTP Verb: DELETE
Caller: Any party involved in the Contract (CONSUMER, PRODUCER, PUBLISHER, MARKETPLACE)
HTTP Output Status Codes: 
    HTTP 202 - Accepted
    HTTP 400 - Bad request
    HTTP 401 - Forbidden  
```

##### Input Parameters

| Parameter | Type  | Description |
|:----------|:------|:------------|
|contractId |string |Id of the contract (URL)|
|actorId    |string |Id of user signing the request, comes from the **HTTP Authorization header** |

If **OCEAN DB** is enabled, the content of the Metadata attribute will be pass as parameter to the Ocean DB implementation to be stored in an external system.  


<a name="contract-output"></a>
##### Output

A complete description of the Contract output can be found in the [previous section](#contract-output).

#### Orchestration Layer

The AGENT node will be in charge of manage the Contracts update. 
The Ocean DB integration is optional, so the Metadata will be stored there only if Ocean DB interface implementation is provided.

The AGENT interact with the `AssetsRegistry::revoke`, allowing to the user sending the request `msg.sender` to revoke the contract signature given previously.

#### Interaction with the Keeper

During the execution of the `revoke` function, the following tasks MUST be implemented:

* Check the authorization of the msg.sender
* Check the contract availability
* Check if the user already signed the contract
* Update the state to **CANCELLED** and remove any previous flag with the user signature
* Remove the associated Consumption Information if already exists


#### Interaction with Ocean DB

The integration with OCEAN DB is optional, so only will works if an implementation backend is provided.

If it's enabled, the Ocean DB layer will interact with the backend to store the metadata information about the contracts. So a part of the information defined in the Output section, the AGENT will forward the information related with the change of state. 




---

<a name="additional-info"></a>
## Additional info

### Assignee(s)
Primary assignee(s): @aaitor


### Targeted Release

The implementation of the full Keeper functionality it's planned for the [Alpha release](https://github.com/oceanprotocol/ocean/milestone/4)


### Status
unstable

<a name="copyright-waiver"></a>
### Copyright Waiver  
To the extent possible under law, the person who associated CC0 with this work has waived all copyright and related or neighboring rights to this work.
