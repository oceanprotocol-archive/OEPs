```text
shortname: 11/ACL
name: On-Chain Access Control using Service Execution Agreements
type: Standard
status: Raw
version: 0.1
editor: Aitor Argomaniz <aitor@oceanprotocol.com>
contributors: Lev Berman <ldmberman@gmail.com>,
              Ahmed Ali <ahmed@oceanprotocol.com>, 
              Samer Sallam <samer@oceanprotocol.com>,
              Dimitri De Jonghe <dimi@oceanprotocol.com>,
              Troy McConaghy <troy@oceanprotocol.com>
```

**Table of Contents**

<!--ts-->
   * [Abstract](#abstract)
   * [Change Process](#change-process)
   * [Language](#language)
   * [Motivation](#motivation)
   * [Actors and Technical Components](#actors-and-technical-components)
   * [Flows](#flows)
      * [Publishing](#publishing)
         * [Constructing an Asset DDO](#constructing-an-asset-ddo)
         * [EscrowAccessSecretStore Service Agreement Template](#escrowaccesssecretstore-service-agreement-template)
      * [Consuming](#consuming)
         * [Execution of the SEA](#execution-of-the-sea)
            * [Lock Payment Condition](#lock-payment-condition)
            * [Grant Access Condition](#grant-access-condition)
            * [Release Payment Condition](#release-payment-condition)
      * [Consuming the Data](#consuming-the-data)
         * [Consuming without direct integration of Secret Store](#consuming-without-direct-integration-of-secret-store)
         * [Abort Conditions](#abort-conditions)
   * [Encryption and Decryption](#encryption-and-decryption)
      * [Using Secret Store](#using-secret-store)
      * [Using BRIZO](#using-brizo)
   * [Implementation Details](#implementation-details)

<!-- Added by: troy, at: 2019-05-20T16:20+02:00 -->

<!--te-->

---

# Abstract

This OEP introduces an integration pattern for the use of **Service Execution Agreements (SEAs)** (also called "Service Agreements" or "Agreements") as contracts between parties interacting in a transaction.
This OEP evolves the existing [OEP-10](../10/README.md), using the SEA as the core element to orchestrate the publish/consume transactions for multiple services.

This OEP doesn't detail the implementation of SEAs; see [the dev-ocean repository](https://github.com/oceanprotocol/dev-ocean).

# Change Process

The process to change this document is described in [OEP-2 (COSS)](../2/README.md).

# Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.

# Motivation

The main motivations of this OEP are:

* Evolve [OEP-10](../10/README.md) to support more complex contract interactions between consumers and publishers of services
* Detail the main characteristics of this interaction
* Introduce an alternative negotiation mechanism to the one described in [OEP-10](../10/README.md)
* List the pros and cons of this approach
* Identify the modifications required to integrate this approach
* Identify the API methods exposed via the different libraries
* Develop a more secure/stable approach overall

# Actors and Technical Components

* PUBLISHERS - Provide access to assets and/or services
* CONSUMERS - Want to get access to assets and/or services
* MARKETPLACES or DOMAINS - Store the DDO (including metadata) associated with the assets and/or services

Note: Below, we write "assets" to mean "assets and/or services."

The following technical components are involved with the publishing flow or the consumption flow:

* MARKETPLACE - Exposes a web interface allowing users to publish and purchase assets. Also facilitates the discovery of assets. An example MARKETPLACE web interface is [Pleuston](https://github.com/oceanprotocol/pleuston).
* [SQUID](https://github.com/oceanprotocol/dev-ocean/blob/master/doc/architecture/squid.md) - Software library encapsulating the Ocean Protocol business logic. It's used to interact with all the components & APIs of the system. It's currently implemented in the following packages:
  * [squid-js](https://github.com/oceanprotocol/squid-js) - JavaScript version of SQUID to be integrated with front-end applications.
  * [squid-py](https://github.com/oceanprotocol/squid-py) - Python version of SQUID to be integrated with back-end applications. The primary users are data scientists.
  * [squid-java](https://github.com/oceanprotocol/squid-java) - Java version of SQUID to be integrated with [JVM](https://en.wikipedia.org/wiki/Java_virtual_machine) applications. The primary users are data engineers.
* [KEEPER CONTRACTS](https://github.com/oceanprotocol/keeper-contracts) - Provide the Service Agreement business logic.
* [SECRET STORE](https://github.com/oceanprotocol/parity-ethereum) - Included as part of the Parity Ethereum client. Allows the PUBLISHER to encrypt the asset URL. Integrates with the SA to authorize (on-chain) the decryption of the asset URL by the CONSUMER.
* [BRIZO](https://github.com/oceanprotocol/brizo) - Microservice to be executed by PUBLISHERS. It exposes an HTTP REST API permitting access to PUBLISHER assets or additional services such as computation.
* [AQUARIUS](https://github.com/oceanprotocol/aquarius) - Microservice to be executed by MARKETPLACES. Facilitates creating, updating, deleting and searching the asset metadata registered by the PUBLISHERS. This metadata is included as part of a DDO (see [OEP-7](../7/README.md) and [OEP-8](../8/README.md)) and also includes the services associated with the asset (consumption, computation, etc.).

![Actors running Components](images/software-run-by-actors.png)

# Flows

This section describes the asset publishing flow and the asset consuming flow in detail. It should be straightforward to implement those flows by reading it, although the actual implementation may deviate slightly.
The detailed description is an attempt to account for important edge cases and to create a good reference for the authors of particular implementations.

The following parameters are used:

* **did** - Decentralized Identifier (DID). See [OEP-7](../7/README.md).
* **agreementId** or **serviceAgreementId** - The unique ID referring to a Service Agreement established between a PUBLISHER and a CONSUMER. The CONSUMER (via SQUID) is the one creating this unique ID.
* **serviceDefinitionId** - Identifies one service in the array of services included in the DDO. It is created by the PUBLISHER (via SQUID) upon DDO creation.
* **templateId** - Identifies a unique Service Agreement template. All supported templates are deployed on-chain by Ocean Protocol and the addresses of templateIds can be found in the ABIs. The following templates are currently supported: `EscrowAccessSecretStoreTemplate`

## Publishing

When a PUBLISHER uses SQUID to publish (register) an asset, here is a summary of what SQUID does:

1. Construct a new DDO (JSON object describing the asset).
1. Register the DDO on-chain through [the DIDRegistry smart contract (KEEPER CONTRACT)](https://github.com/oceanprotocol/keeper-contracts/tree/develop/contracts/registry).
1. Store the DDO off-chain in an AQUARIUS instance's database.

Those steps are explained in more detail below.

The PUBLISHER should be able to publish (register) an asset by making a single SQUID call. For example, the squid-py call might look like:

```python
ddo = ocn.assets.create(metadata, publisher_account)
```

where `metadata` is a dict containing the [OEP-8](../8/README.md) metadata about the asset.

To see _exactly_ what the asset-publishing SQUID call does, see the implementations in:

* [squid-js: the OceanAssets.create() method](https://github.com/oceanprotocol/squid-js/blob/develop/src/ocean/OceanAssets.ts) (TypeScript)
* [squid-py: the OceanAssets.create() method](https://github.com/oceanprotocol/squid-py/blob/develop/squid_py/ocean/ocean_assets.py)
* [squid-java: the OceanManager.registerAsset() method](https://github.com/oceanprotocol/squid-java/blob/develop/src/main/java/com/oceanprotocol/squid/manager/OceanManager.java) (The Assets API `create()` method calls that.)

We now expand on the publishing (registration) steps in more detail.

### Constructing an Asset DDO

An asset DDO is a [DID Document](https://w3c-ccg.github.io/did-spec/#did-documents) conforming with [the Decentralized Identifiers (DIDs) spec](https://w3c-ccg.github.io/did-spec/).

1. Validate the metadata to ensure that it conforms with [OEP-8](../8/README.md). (It should be in "local metadata" form at this point.)
1. Compute a DID following [OEP-7](../7/README.md).
1. Create an empty DDO and add the following things to it:
   * DID
   * Public key of the PUBLISHER
   * Authentication section (with RSA public key)
1. Encrypt the URLs in the `base.files` array of the metadata.
   The PUBLISHER must specify which encryption service/procedure/plugin they wish to use.
   That encryption service gets recorded in the asset DDO.
   For details, see the section about [Encryption and Decryption](#encryption-and-decryption) below.
   Note: This step changes the metadata and also the `"service"` section of the DDO.
1. Compute the checksum of the metadata according to [OEP-8](../8/README.md). Add the `base.checksum` key and value to the metadata.
1. Sign that checksum using the `publisher_account` (i.e. compute a signature) and add the computed signature to the `proof` attribute.
1. Add the rest of the services to the DDO.
   Each service in the list contains certain information depending on its type. Here we document two types of services required for purchasing and consuming an asset.

   A service of type "Access" contains:

   * Service Definition ID (`serviceDefinitionId`); this helps PUBLISHER find the service definition of a DDO signed by CONSUMER
   * Service Agreement Template ID (`templateId`); has to be whitelisted, can be hardcoded in Trilobite; points to a deployed on-chain Service Agreement contract
   * Service endpoint (`serviceEndpoint`); CONSUMERS signing this service send their signatures to this endpoint
   * A list of condition keys; condition key is the `keccak256` hash of the following:
     * SLA template ID
     * controller contract address (obtained from the solidity contract json file matching the contract name in the SLA condition)
     * controller contract function fingerprint (referred to as function signature or selector)

    ```python
    def build_condition_key(contract_address, fingerprint, template_id):
        assert isinstance(fingerprint, bytes), f'Expecting `fingerprint` of type bytes, ' \
            f'got {type(fingerprint)}'
        return generate_multi_value_hash(
            ['address', 'address', 'bytes4'],
            [template_id, contract_address, fingerprint]
        ).hex()
    ```

   * For each condition, a list is required of its parameter values, a timeout, a set of fields determining what conditions depend on other conditions, and a mapping of events emitted by the condition to the off-chain handlers of these events
   * Each event is identified by name. Each event handler is a function from a whitelisted module
   * Service Agreement contract address and the event mapping in the same format as the condition events, for off-chain listeners
   * An integer defining when the agreement is fulfilled in case there are multiple terminal conditions, according to the Service Agreement smart contract

   A service of type "Access" contains 2 different endpoints:

   * **serviceEndpoint** - A URL to initialize the Service Agreement
   * **consumeEndpoint** - A URL to fetch data decryption keys from

    An example of a complete DDO can be found [here](./ddo.example.json). Please do note that the condition's order in the DID document should reflect the same order in on-chain service agreement.

1. PUBLISHER registers the DID, associating the asset DID to the Aquarius Metadata URL that resolves the DID to a DDO.
To do that, SQUID needs to integrate the `DIDRegistry` contract using the `registerAttribute` method.

```javascript
function registerAttribute (
    bytes32 _did,
    bytes32 _checksum,
    address[] memory _providers,
    string memory _value
)
```

The parameters to pass are:

  * **bytes32 _did** - The hash part of the DID, the part just after `did:op:`
  * **bytes32 _checksum** - The checksum generated after [compute the DID](https://github.com/oceanprotocol/OEPs/tree/master/7#how-to-compute-a-did)
  * **address[] _providers** - The list of providers which PUBLISHER delegates URL decryption capabilities and SEA management
  * **string _value** - The Metadata service endpoint. In the above DDO its: http://myaquarius.org/api/v1/provider/assets/metadata/{did}

![Publishing Flow](images/publishing-flow.png)

1. The KEEPER will emit the `DIDAttributeRegistered` including the `did`, `checksum` and `url` registered.

### EscrowAccessSecretStore Service Agreement Template

1. The EscrowAccessSecretStore Service Agreement template has the following shape:

```javascript
const agreement = {
    did: did,
    conditionIds: [
        conditionIdAccess,
        conditionIdLock,
        conditionIdEscrow
    ],
    timeLocks: [timeLockAccess, 0, 0],
    timeOuts: [timeOutAccess, 0, 0],
    consumer: receiver
}
```

1. For the different conditionIds, the CONSUMER needs to generate those and add them to the agreement to be defined on-chain.
   This requires to generate the hash including the **agreementId** and all the values of the specific condition:

```javascript
const conditionIdAccess = await accessSecretStoreCondition.generateId(agreementId, await accessSecretStoreCondition.hashValues(did, receiver))
const conditionIdLock = await lockRewardCondition.generateId(agreementId, await lockRewardCondition.hashValues(escrowReward.address, escrowAmount))
const conditionIdEscrow = await escrowReward.generateId(agreementId, await escrowReward.hashValues(escrowAmount, receiver, sender, conditionIdLock, conditionIdAccess))
```

1. PUBLISHER publishes the DDO in the Metadata Store (OceanDB) using AQUARIUS.

## Consuming

Using SQUID calls, a CONSUMER can discover, purchase and get access to assets.

Steps for leveraging SQUID:

1. The CONSUMER uses the search method to find relevant assets related with his query. It returns a list of DDO's.
   `assets = ocean.assets.search("weather Germany 2017")`

1. The CONSUMER chooses a service inside a DDO (the CONSUMER selects a `serviceDefinitionId`).

1. The Service Agreement needs to have an associated unique `serviceAgreementId` that can be generated/provided by the CONSUMER.    
   In the Smart Contracts, this `serviceAgreementId` will be stored as a `bytes32`. This `serviceAgreementId` is random and is represented by a 64-character hex string (using the characters 0-9 and a-f).
   The CONSUMER can generate the `serviceAgreementId` using any kind of implementation providing enough randomness to generate this ID (64-characters hex string).

1. The CONSUMER signs the service details. The signature contains `(templateId, valuesHashList, timeoutValues, agreementId)`. 
   The `agreementId` is provided by the CONSUMER and has to be globally unique.
   * Each ith item in `values_hash_list` lists corresponds to the ith condition in conditions list
   * `values_hash_list`: a hash of the parameters types and values of each condition
```python
def create_condition_params_hash(parameters_types, parameters_values):
    return web3.Web3.soliditySha3(parameters_types, parameters_values).hex()
    
create_condition_params_hash(['bytes32', 'uint256'], ['0x...', '25'])
```
 
```python
def generate_service_agreement_hash(template_id, values_hash_list, timelocks, timeouts, agreement_id):
    return web3.soliditySha3(
            ['address', 'bytes32[]', 'uint256[]', 'uint256[]', 'bytes32'],
            [template_id, values_hash_list, timelocks, timeouts, agreement_id]
    )
# Sign the agreement hash
web3_instance.eth.sign(address, generate_service_agreement_hash(...))

#The content of value_hash_list is generated calling this method:
    def generate_agreement_condition_ids(self, agreement_id, asset_id, consumer_address,
                                         publisher_address, keeper):
        lock_cond_id = keeper.lock_reward_condition.generate_id(
            agreement_id,
            self.condition_by_name['lockReward'].param_types,
            [keeper.escrow_reward_condition.address, self.get_price()]).hex()

        access_cond_id = keeper.access_secret_store_condition.generate_id(
            agreement_id,
            self.condition_by_name['accessSecretStore'].param_types,
            [asset_id, consumer_address]).hex()

        escrow_cond_id = keeper.escrow_reward_condition.generate_id(
            agreement_id,
            self.condition_by_name['escrowReward'].param_types,
            [self.get_price(), publisher_address, consumer_address,
             lock_cond_id, access_cond_id]).hex()

        return access_cond_id, lock_cond_id, escrow_cond_id
```

This signature is used to correlate events and to prevent the PUBLISHER from instantiating multiple Service Agreements from a single request.


1. The CONSUMER sends `(did, serviceAgreementId, serviceDefinitionId, signature, consumerAddress`) to the service endpoint (BRIZO).
`serviceDefinitionId` tells the PUBLISHER where to find the preimage to verify the signature. The DID tells the PUBLISHER which asset to serve under these terms.

```
HTTP POST /api/v1/brizo/services/access/initialize

{
 "did": "did:op:08a429b8529856d59867503f8056903a680935a76950bb9649785cc97869a43d",
 "serviceAgreementId": "bb23s87856d59867503f80a690357406857698570b964ac8dcc9d86da4ada010",
 "serviceDefinitionId": "0",
 "signature": "cade376598342cdae231321a0097876aeda656a567a67c6767fd8710129a9dc1",
 "consumerAddress": "0x00a329c0648769A73afAc7F9381E08FB43dBEA72"
}

```

The execution of this endpoint should return a `HTTP 201` if everything goes okay. Satisfactory conditions include:

   - When BRIZO receives a signature from the service endpoint and verifies the signature.

   - Having the `did`, BRIZO fetches the DDO related with this `did`.

   - BRIZO records the `serviceAgreementId` as corresponding to the given `did`.

   - BRIZO executes the Service Agreement by calling `EscrowAccessSecretStoreTemplate.createAgreement`, providing it with the agreementId and all the agreement values

   - BRIZO starts listening for the `publisher` events from the events section of the service definition.

1. After receiving the HTTP response confirmation from BRIZO, the CONSUMER starts listening for the `AgreementCreated` events specified in the corresponding service definition, filtering them by `agreementId`.

### Execution of the SEA

Consider an asset purchase example. CONSUMER locks the payment. Then PUBLISHER grants access to the document. Then payment is released. Now CONSUMER may decrypt the document.

In general, there is a broad range of conditions which can be implemented and integrated into the described workflow.

#### Lock Payment Condition

Consider a sample of a service definition.

```
"serviceAgreementTemplate": {
		"contractName": "EscrowAccessSecretStoreTemplate",
		"events": [{
			"name": "AgreementCreated",
			"actorType": "consumer",
			"handler": {
				"moduleName": "escrowAccessSecretStoreTemplate",
				"functionName": "fulfillLockRewardCondition",
				"version": "0.1"
			}
		}]
  }
```

According to this sample, the CONSUMER listens for the `AgreementCreated` event emitted in the very beginning of Service Agreement execution, filtering it by `agreementId`.

Note that the structure of `serviceAgreementContract.events` is identical to `conditions.events`. SQUID needs to offer a utility that subscribes the specified callbacks to the events from both lists.

When the CONSUMER receives this event it means the agreement is in place and can perform the lock reward: 

```
await oceanToken.approve(lockRewardCondition.address, escrowAmount, { from: sender })
await lockRewardCondition.fulfill(agreementId, escrowReward.address, escrowAmount)
```

If everything goes right, it will emit `LockRewardCondition.Fulfilled` and thus will trigger the next condition. 

#### Grant Access Condition

PUBLISHER (via BRIZO) listens for `LockRewardCondition.Fulfilled` event filtered by `agreementId` to confirm the reward was locked by the CONSUMER.

```
"conditions": [{
  "events": [{
        "name": "Fulfilled",
        "actorType": "publisher",
        "handler": {
            "moduleName": "lockRewardCondition",
            "functionName": "fulfillAccessSecretStoreCondition",
            "version": "0.1"
        }
    }]
}]
```

In this case the PUBLISHER can grant access to the CONSUMER for a specific `agreementId` and `documentId` using in this case the `AccessSecretStoreCondition.fulfill`:

```
await accessSecretStoreCondition.fulfill(agreementId, agreement.did, receiver)
```

If everything goes right, the Smart Contract will emit the `AccessSecretStoreCondition.Fulfilled` event.

#### Release Payment Condition

PUBLISHER (via BRIZO) listens for `AccessSecretStoreCondition.Fulfilled` event to transfer tokens to PUBLISHER's account.

```
"conditions": [{
    "events": [{
        "name": "Fulfilled",
        "actorType": "publisher",
        "handler": {
            "moduleName": "accessSecretStore",
            "functionName": "fulfillEscrowRewardCondition",
            "version": "0.1"
        }
    }]
}]
```

So when the PUBLISHER receives the `AccessSecretStoreCondition.Fulfilled` he can call the `EscrowReward.fulfill` method to receive the reward:

```
await escrowReward.fulfill(agreementId, escrowAmount, receiver, sender, agreement.conditionIds[1], agreement.conditionIds[0])
```

## Consuming the Data

CONSUMER (via SQUID) listens for `AccessSecretStoreCondition.Fulfilled` event to access the document.

```
"conditions": [{
    "events": [{{
        "name": "TimedOut",
        "actorType": "consumer",
        "handler": {
            "moduleName": "accessSecretStore",
            "functionName": "fulfillEscrowRewardCondition",
            "version": "0.1"
        }
    }]
}]
```

The following are steps that have to be performed by the CONSUMER to receive the data.

1. CONSUMER decrypts the URL using SQUID. This only requires the encryptedUrl existing in the DDO and the DID. 
   A Parity EVM client (local or remote) and SECRET STORE cluster can be used for that.

1. CONSUMER retrieves data by calling the dedicated BRIZO endpoint (`serviceEndpoint` in the service definition) 
providing it with Consumer ethereum address, service agreement ID, and decrypted URL.

The consume URL may look like:

```
HTTP GET /api/v1/brizo/services/access/consume?consumerAddress=${consumerAddress}&serviceAgreementId={serviceAgreementId}&url={url}`
```

This method will return a HTTP 200 status code if everything was okay and the data file.

When CONSUMER requests purchased data, BRIZO gets 3 parameters:

* Consumer ethereum address: `consumerAddress`
* Service Agreement ID: `serviceAgreementId`
* Decrypted URL: `url`. This URL is only valid if BRIZO acts as a proxy. CONSUMER cannot download using the URL if it's not done through BRIZO.

Using those parameters, BRIZO does the following things:

* Find the `did` by the given `serviceAgreementId`

* Verify the given service is allowed to be consumed by the given `consumerAddress` and `did` using the `checkPermissions` method of the `SLA` Smart Contract.

* If CONSUMER has permissions to consume, download and provide data for the given DID

![Consuming Flow](images/consuming-flow.png)

### Consuming without direct integration of Secret Store

If the CONSUMER (via SQUID) can't integrate directly SECRET STORE for decryption (squid-js using Metamask can't provide the account password), 
it's possible to call BRIZO with an alternative `consume` method.

In this scenario, it's BRIZO as provider the one in charge of decrypting the content in behalf of the CONSUMER.

The consume URL may look like:

```
HTTP GET /api/v1/brizo/services/access/consume?pubKey=${pubKey}&serviceAgreementId={serviceAgreementId}&signature={signature}&index={index}`
```

This method will return an HTTP 200 status code if everything was okay, plus the URL required to get access to the data.

When CONSUMER requests purchased data, BRIZO gets 3 parameters:

* Consumer public key: `pubKey`
* Service Agreement ID: `serviceAgreementId`
* Signature: `signature`. The signed `serviceAgreementId` value by the CONSUMER to validate his/her identity
* Index: `index`. Integer value representing the position of the content to download in the `DDO.files` array

### Abort Conditions

Every condition can be fulfilled or aborted using the configured timeout.
For example it would allows to the CONSUMER to cancel the payment after locking it but not receiving access to the asset for a long period of time.
Mechanisms implemented in the Service Agreement contract ensure there are no race conditions.

# Encryption and Decryption

The PUBLISHER can define how they want to encrypt the URLs in the `base.files` array of the metadata. This information must be added to the DDO to allow CONSUMERs (via SQUID) to understand how to deal with the URLs. Below is an example of how to add an encryption service to the `service` section of a DDO.

```json
"service": [{
    "type": "Authorization",
    "service": "SecretStore",
    "serviceDefinitionId": "0",
    "serviceEndpoint": "http://secretstore.org:12001"
  },
  …
]
```

The encryption service is one object with the following attributes:

* type - Differentiate this kind of service with the word **Authorization**
* service - The authorization service type.
* serviceDefinitionId - Existing in all the DDO services to differentiate one entry in the `services` list
* serviceEndpoint (optional) - URL used during the encryption and decryption process.

The encryption/authorization service is optional. If it's not provided, the usual SECRET STORE cluster defined in the SQUID configuration will be used.

We now describe the supported encryption procedures.

## Using Secret Store

The SECRET STORE cluster to use during the encryption and decryption is specified in the **serviceEndpoint** attribute, e.g.

```json
"service": [{
    "type": "Authorization",
    "service": "SecretStore",
    "serviceDefinitionId": "0",
    "serviceEndpoint": "http://secretstore.org:12001"
  },
  …
]
```

Suppose the `base.files` array in the metadata has three URLs:

```json
"files": [
    {
        "url": "https://example.com/data-file-0.csv",
        "index": 0,
        "checksum": "efb2c764274b745f5fc37f97c6b0e761",
        "contentLength": "4535431",
        "resourceId": "access-log2018-02-13-15-17-29-18386C502CAEA932"
    },
    {
        "url": "https://example.com/data-file-1.csv",
        "index": 1,
        "checksum": "085340abffh21495345af97c6b0e761",
        "contentLength": "12324"
    },
    {
        "url": "https://example.com/data-file-2.csv",
        "index": 2
    }
]
```

The `base.files` array is encrypted as follows.
First it is converted into a string like so:

```json
[{"url":"https://example.com/data-file-0.csv","index":0,…,"index":2}]
```

where all spaces are removed (except inside the string values). Also, all newlines, line feeds, and carriage returns are removed. That JSON string can then be encrypted.

After encryption, all `"url"` keys and values are removed from the `base.files` array objects, and a new `base.encryptedFiles` key and value are added to the metadata, e.g.

```json
"encryptedFiles": "0x2e48ceefcca7abb024f90…f3fec0e1c"
```

More information about the integration of a SECRET STORE can be found [in the dev-ocean repository](https://github.com/oceanprotocol/dev-ocean/blob/master/doc/architecture/secret-store.md).

## Using BRIZO

For those clients not able to integrate SECRET STORE directly, BRIZO will support an encryption endpoint supporting the following parameters:

```http
HTTP POST /api/v1/brizo/services/encrypt

{
 "id": "did:op:08a429b8529856d59867503f8056903a680935a76950bb9649785cc97869a43d",
 "document": [
     {
         "url": "234ab87234acbd09543085340abffh21983ddhiiee982143827423421",
         "checksum": "efb2c764274b745f5fc37f97c6b0e761",
         "contentLength": "4535431",
         "resourceId": "access-log2018-02-13-15-17-29-18386C502CAEA932"
      },
      {
          "url": "234ab87234acbd6894237582309543085340abffh21983ddhiiee982143827423421",
          "checksum": "085340abffh21495345af97c6b0e761",
          "contentLength": "12324"
      },
      {
          "url":"80684089027358963495379879a543085340abffh21983ddhiiee982143827abcc2"
      }
  ]
}
```

That is, the value of `document` should be the `base.files` array.

This endpoint will return the content encrypted using the BRIZO account.

# Implementation Details

- SQUID: An utility to compute CONSUMER signatures.

    ```
    signature = sign(service)
    ```

- SQUID, BRIZO: A generic event handler which:
   * listens for the given web3 events
   * filters them by the given service ID
   * calls provided event handlers, passing them event payload and given service definition

- Event handlers:
   * SQUID: payments : lock payment
   * BRIZO: payment : release payment
   * BRIZO: SECRET STORE : grant access
   * SQUID: consumer: retrieve data

- SQUID: A function for cancelling payments
- SQUID: A function for publishing an asset
- SQUID: A function for purchasing an asset, which consists of:
   * signing the given service
   * registering service ID locally
   * calling the purchase endpoint
   * subscribing to events

- BRIZO: An endpoint for accepting purchases and instantiating service agreements
- BRIZO: Consume endpoints for providing decrypted keys and purchased data
