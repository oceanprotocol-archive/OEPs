```
shortname: 11/ACL
name: On-Chain Access Control using Service Agreements
type: Standard
status: Raw
editor: Aitor Argomaniz <aitor@oceanprotocol.com>
contributors: Lev Berman <ldmberman@gmail.com>,
			  Ahmed Ali <ahmed@oceanprotocol.com>, 
              Samer Sallam <samer@oceanprotocol.com>,
              Dimitri De Jonghe <dimi@oceanprotocol.com>

```

Table of Contents
=================

   * [Table of Contents](#table-of-contents)
   * [On-Chain Access Control using Service Agreements](#on-chain-access-control-using-service-agreements)
      * [Change Process](#change-process)
      * [Language](#language)
      * [Motivation](#motivation)
      * [Actors](#actors)
         * [Technical components](#technical-components)
      * [Flow](#flow)
         * [Publishing](#publishing)
         * [EscrowAccessSecretStore Service Agreement Template](#escrowaccesssecretstore-service-agreement-template)
         * [Consuming](#consuming)
            * [Execution of the SEA](#execution-of-the-sea)
               * [Lock Payment Condition](#lock-payment-condition)
               * [Grant Access Condition](#grant-access-condition)
               * [Release Payment Condition](#release-payment-condition)
         * [Consuming the Data](#consuming-the-data)
            * [Abort Conditions](#abort-conditions)
         * [Encryption and Decryption](#encryption-and-decryption)
            * [No Encryption](#no-encryption)
            * [Secret Store](#secret-store)
            * [Rsa Public and Private Keys](#rsa-public-and-private-keys)
         * [Implementation details](#implementation-details)



---


# On-Chain Access Control using Service Agreements

This OEP introduces the integration pattern for the use **Service Execution Agreements (SEA)** as contracts between parties interacting in a transaction.
This OEP evolves the existing OEP-10 [OEP-10](../10/README.md), using the SEA as the core element to orchestrate the publish/consume transactions for multiple services.

It's out of the scope to detail the Service Agreements implementation. Service Agreements are described as part of the Dev-Ocean repository.

## Change Process

This document is governed by the [2/COSS](../2/README.md) (COSS).


## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.


## Motivation

The main motivations of this OEP are:

* Evolve the OEP-10 to support more complex contract interactions between consumers and publishers of services
* Detail the main characteristics of this interaction
* Introduce an alternative negotiation mechanism to the one described in the [OEP-10](../10/README.md)
* Specify the pros and cons of this approach
* Identify the modifications required to integrate this approach
* Identify the API methods exposed via the different libraries
* Develop a more secure/stable approach overall


## Actors

The different actors interacting in this flow are:

* PUBLISHERS - Provide access to Assets and/or Services
* CONSUMERS - Want to get access to Assets or Services
* MARKETPLACES or DOMAINS - Store the DDO/Metadata related with the Assets

### Technical components

The following technical components are involved in an end-to-end publishing and consumption flow:

* [MARKETPLACE](https://github.com/oceanprotocol/pleuston) - Exposes a web interface allowing the users to publish and purchase assets. Also facilitates the discovery of assets.
* SQUID - Library encapsulating the Ocean Protocol business logic. Interacts with all the different components/APIs of the system. Currently it's provided in the following languages:
  - [Squid Javascript](https://github.com/oceanprotocol/squid-js) - Javascript version of Squid to be integrated with Frontend applications.
  - [Squid Python](https://github.com/oceanprotocol/squid-py) - Python version of Squid to be integrated with Backend applications. The primary users are data scientists.
  - [Squid Java](https://github.com/oceanprotocol/squid-java) - Java version of Squid to be integrated with JVM applications. The primary users are data engineers.
* [KEEPER CONTRACTS](https://github.com/oceanprotocol/keeper-contracts) - Provides the Service Agreement (SA) business logic.
* [SECRET STORE](https://github.com/oceanprotocol/parity-ethereum) - Included as part of the Parity Ethereum client. Allows the PUBLISHER to encrypt the Asset url. Integrates with the SA in order to authorize on-chain the decryption of the Asset url by the CONSUMER
* [BRIZO](https://github.com/oceanprotocol/brizo) - Microservice to be executed by a PUBLISHER. It exposes the HTTP REST API permitting access to PUBLISHER Assets or additional services like computation.
* [AQUARIUS](https://github.com/oceanprotocol/aquarius) - Microservice to be executed by the MARKETPLACES. Facilitates creating, updating, deleting and searching the Asset's metadata registered by the PUBLISHERS. This Metadata, is included as part of a [DDO](../7/README.md), which also includes the Services associated with the Asset (Consumption, Computation, etc.).

![Actors running Components](images/software-run-by-actors.png)

## Flow

This section describes the Asset purchase flow in detail. It should be straightforward to implement the flow by reading this description, although the actual implementation may deviate slightly.
The detailed description is an attempt to account for important edge cases and to create a good reference for the authors of particular implementations.

There are some parameters used in this flow:

* **did** - See [OEP-7](../7/README.md).
* **agreementId** or **serviceAgreementId** - Is the unique ID referring to a Service Agreement established between a PUBLISHER and a CONSUMER. The CONSUMER (via Squid) is the one creating this random unique serviceId.
* **serviceDefinitionId** - Identifies one service in the array of services included in the DDO. It is created by the PUBLISHER (via Squid) upon DDO creation and is associated with different services.
* **templateId** - Identifies a unique Service Agreement template. The templates supported are deployed on-chain by Ocean Protocol and the addresses or templateId's can be found in the ABI's. Initially the following templates is supported:
  - **EscrowAccessSecretStoreTemplate**


### Publishing

Using only one Squid call, the PUBLISHER should be able to register an Asset:
```
const asset = ocean.assets.create(metadata, publisherAccount, services=[ocean.services.createAccessService(...)])
```

This method executes internally:

1. PUBLISHER generates a DID. See [How to generate a DID](https://github.com/oceanprotocol/OEPs/tree/master/7#length-of-a-did). Currently the DID is a UUID. Later on, this might be computed as a DDO hash.
1. PUBLISHER optionally can encrypt the URLs using different encryption plugins. If that's the case, in the DDO will be added an **encryption** service describing the procedure used.
1. PUBLISHER encrypts the URL using the Secret Store identified by a DID.
1. PUBLISHER creates a DDO including the following information:
   - DID
   - Metadata
     Contains Asset name, description, etc. For details see [OEP-8](../8/README.md).
   - Public key of the PUBLISHER
   - Encrypted URL; this URL, when decrypted, is not used by the CONSUMER directly, but as an identifier to request data from PUBLISHER
   - A list of services (Access, Compute, etc)

   Each service in the list contains certain information depending on its type. Here we document two types of services required for purchasing and consuming an Asset. 

   A service of type "Access" contains:
   - Service Definition ID (`serviceDefinitionId`); this helps PUBLISHER find the service definition of a DDO signed by CONSUMER
   - Service Agreement Template ID (`templateId`); has to be whitelisted, can be hardcoded in Trilobite; points to a deployed on-chain Service Agreement contract
   - Service endpoint (`serviceEndpoint`); CONSUMERS signing this service send their signatures to this endpoint
   - A list of condition keys; condition key is the `keccak256` hash of the following:
     * SLA template ID
     * controller contract address (obtained from the solidity contract json file matching the contract name in the SLA condition)
     * controller contract function fingerprint (referred to as function signature or selector)

```
def generate_condition_key(sla_template_id, contract_address, function_fingerprint):
    key = web3.Web3.soliditySha3(
        ['bytes32', 'address', 'bytes4'],
        [sla_template_id.encode(), contract_address, function_fingerprint]
    )
    return key.hex()
    
```

   - For each condition, a list is required of its parameter values, a timeout, a set of fields determining what conditions depend on other conditions, and a mapping of events emitted by the condition to the off-chain handlers of these events
   - Each event is identified by name. Each event handler is a function from a whitelisted module
   - Service Agreement contract address and the event mapping in the same format as the condition events, for off-chain listeners
   - An integer defining when the agreement is fulfilled in case there are multiple terminal conditions, according to the Service Agreement smart contract

   A service of type "Access" contains 2 different endpoints:
   - **serviceEndpoint** - A URL to fetch data decryption keys from
   - **purchaseEndpoint** - A URL to initialize the Service Agreement

    An example of a complete DDO can be found [here](./ddo.example.json). Please do note that the condition's order in the DID document should reflect the same order in on-chain service agreement.

1. PUBLISHER compute the key attributes of the DDO, generate a `checksum` and sign. The signature is added in the `proof` attribute.
   More information about this process can be found in the [How to Compute a DID section](https://github.com/oceanprotocol/OEPs/tree/master/7#how-to-compute-a-did).

1. PUBLISHER publishes the DDO in the Metadata Store (OceanDB) using AQUARIUS.

1. PUBLISHER registers the DID, associating the Asset DID to the Aquarius Metadata URL that resolves the DID to a DDO.
To do that, SQUID needs to integrate the `DIDRegistry` contract using the `registerAttribute` method.

```
function registerAttribute (
        bytes32 _did,
        bytes32 _checksum,
        string memory _value
    )
```

The parameters to pass are:
  - **bytes32 _did** - The hash part of the DID, the part just after `did:op:`
  - **bytes32 _checksum** - The checksum generated after [compute the DID](https://github.com/oceanprotocol/OEPs/tree/master/7#how-to-compute-a-did)
  - **string _value** - The Metadata service endpoint. In the above DDO its: http://myaquarius.org/api/v1/provider/assets/metadata/{did}

![Publishing Flow](images/publishing-flow.png)

1. The KEEPER will emit the `DIDAttributeRegistered` including the `did`, `checksum` and `url` registered.


### EscrowAccessSecretStore Service Agreement Template

1. The EscrowAccessSecretStore Service Agreement template has the following shape:

```
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

1. For the different conditionIds, the CONSUMER need to generate those and add to the agreement to be defined on-chain.
   This requires to generate the hash including the **agreementId** and all the values of the specific condition:
   
```   
const conditionIdAccess = await accessSecretStoreCondition.generateId(agreementId, await accessSecretStoreCondition.hashValues(did, receiver))
const conditionIdLock = await lockRewardCondition.generateId(agreementId, await lockRewardCondition.hashValues(escrowReward.address, escrowAmount))
const conditionIdEscrow = await escrowReward.generateId(agreementId, await escrowReward.hashValues(escrowAmount, receiver, sender, conditionIdLock, conditionIdAccess))
```


### Consuming

Using Squid calls, a CONSUMER can discover, purchase and get access to Assets.

Steps for leveraging Squid:

1. The CONSUMER uses the search method to find relevant Assets related with his query. It returns a list of DDO's.
   `assets = ocean.assets.search("weather Germany 2017")`

1. The CONSUMER chooses a service inside a DDO (the CONSUMER selects a `serviceDefinitionId`).

1. The Service Agreement needs to have an associated unique `serviceAgreementId` that can be generated/provided by the CONSUMER.    
   In the Smart Contracts, this `serviceAgreementId` will be stored as a `bytes32`. This `serviceAgreementId` is random and is represented by a 64-character hex string (using the characters 0-9 and a-f).
   The CONSUMER can generate the `serviceAgreementId` using any kind of implementation providing enough randomness to generate this ID (64-characters hex string).

1. The CONSUMER signs the service details. The signature contains `(templateId, valuesHashList, timeoutValues, serviceAgreementId)`. 
   The `serviceAgreementId` is provided by the CONSUMER and has to be globally unique.
   * Each ith item in `values_hash_list` lists corresponds to the ith condition in conditions list
   * `values_hash_list`: a hash of the parameters types and values of each condition
```
def create_condition_params_hash(parameters_types, parameters_values):
    return web3.Web3.soliditySha3(parameters_types, parameters_values).hex()
    
create_condition_params_hash(['bytes32', 'uint256'], ['0x...', '25'])
```
 
```
def generate_service_agreement_hash(web3, sla_template_id, values_hash_list, service_agreement_id):
    return web3.soliditySha3(
        ['bytes32', 'bytes32[]', 'uint256[]', 'bytes32'],
        [sa_template_id, values_hash_list, timeouts, service_agreement_id]
    )
# Sign the agreement hash
web3_instance.eth.sign(address, generate_service_agreement_hash(...))
```

This signature is used to correlate events and to prevent the PUBLISHER from instantiating multiple Service Agreements from a single request.


1. The CONSUMER sends `(did, serviceAgreementId, serviceDefinitionId, signature, consumerAddress`) to the service endpoint (BRIZO).
`serviceDefinitionId` tells the PUBLISHER where to find the preimage to verify the signature. The DID tells the PUBLISHER which Asset to serve under these terms.

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

1. After receiving the HTTP response confirmation from BRIZO, the CONSUMER starts listening for the `AgreementCreated` events specified in the corresponding service definition, filtering them by `serviceAgreementId`.



#### Execution of the SEA

Consider an Asset purchase example. CONSUMER locks the payment. Then PUBLISHER grants access to the document. Then payment is released. Now CONSUMER may decrypt the document.

In general, there is a broad range of conditions which can be implemented and integrated into the described workflow.

##### Lock Payment Condition

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

Note that the structure of `serviceAgreementContract.events` is identical to `conditions.events`. Squid needs to offer a utility that subscribes the specified callbacks to the events from both lists.

When the CONSUMER receives this event means the agreement is in place and can perform the lock reward: 

```
await oceanToken.approve(lockRewardCondition.address, escrowAmount, { from: sender })
await lockRewardCondition.fulfill(agreementId, escrowReward.address, escrowAmount)
```

If everything goes right, it will emits `LockRewardCondition.Fulfilled` and thus triggers the next condition. 

##### Grant Access Condition

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

In that case the PUBLISHER can grant access to the CONSUMER for a specific `agreementId` and `documentId` using in this case the `AccessSecretStoreCondition.fulfill`:

```
await accessSecretStoreCondition.fulfill(agreementId, agreement.did, receiver)
```

If everything goes right, it will emits the `AccessSecretStoreCondition.Fulfilled` event. 



##### Release Payment Condition

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

So when the PUBLISHER received the `AccessSecretStoreCondition.Fulfilled` can call the `EscrowReward.fulfill` method to receive the reward:

```
await escrowReward.fulfill(agreementId, escrowAmount, receiver, sender, agreement.conditionIds[1], agreement.conditionIds[0])
```

### Consuming the Data

CONSUMER (via Squid) listens for `AccessSecretStoreCondition.Fulfilled` event to access the document.

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

1. CONSUMER decrypts the URL using Squid. This only requires the encryptedUrl existing in the DDO and the DID. 
   A Parity EVM client (local or remote) and Secret Store cluster can be used for that.

1. CONSUMER retrieves data by calling the dedicated BRIZO endpoint providing it with CONSUMER public key, service ID, and decrypted URL.

The consume URL may look like:

```
HTTP GET /api/v1/brizo/services/access/consume?pubKey=${pubKey}&serviceAgreementId={serviceAgreementId}&url={url}`
```

This method will return an HTTP 200 status code if everything was okay, plus the URL required to get access to the data.

When CONSUMER requests purchased data, BRIZO gets 3 parameters:

* Consumer public key: `pubKey`
* Service Agreement ID: `serviceAgreementId`
* Decrypted URL: `url`. This URL is only valid if BRIZO acts as a proxy. CONSUMER cannot download using the URL if it's not done through BRIZO.

Using those parameters, BRIZO does the following things:

* Find the `did` by the given `serviceAgreementId`

* Verify the given service is allowed to be consumed by the given `pubKey` and `did` using the `checkPermissions` method of the `SLA` Smart Contract.

* If CONSUMER has permissions to consume, download and provide data for the given DID

![Consuming Flow](images/consuming-flow.png)


#### Abort Conditions

Every condition can be fulfilled or aborted configured timeout. 
For example it would allows to the CONSUMER to cancel the payment after locking it but not receiving access to the Asset for a long period of time. 
Mechanisms implemented in the Service Agreement contract ensure there are no race conditions.



### Encryption and Decryption

The PUBLISHER can define if he/she wants to encrypt or not the URLs before adding to the DDO. This information added to the DDO allows to the CONSUMER's (via SQUID) to understand how to deal with the URLs.
To support this, in the Services section of the DDO can be specified this configuration as a DDO service. Example:

```json
service": [{    
    "type": "Authorization",    
    "service": "SecretStore",
    "serviceDefinitionId": "0",
    "serviceEndpoint": "http://secretstore.org:12001"
  },
  
  {"type": "Access"},
  {"type": "Metadata"}
]
```

This new Service encapsulate one object with the following attributes:

* type - Diferenciate this kind of service with the word **Authorization**
* service - The authorization service type. It could be SecretStore, RSAES-OAEP, None.
* serviceEndpoint (optional) - Url used during the encryption and decryption process.
* serviceDefinitionId - Existing in all the DDO services to differenciate one entry in the `services` list

The authorization service is optional. If it's not provide the usual secret store cluster defined in the SQUID configuration will be used.

The different encryption procedures supported are:

#### No Encryption

This is the case when PUBLISHER doesn't want to encrypt the URLs. This is represented in DDO when:

* The encryption attribute is not defined as part of the service
* The encryption type is **"None"**

Example:
```json
service": [{    
    "type": "Authorization",    
    "service": "None",
    "serviceDefinitionId": "0"
  },
```

#### Secret Store

This is the case when PUBLISHER wants to encrypt the URLs using a Secret Store cluster.
The cluster to use during the encryption and decryption is specfied in the **serviceEndpoint** attribute.

Example:
```json
service": [{    
    "type": "Authorization",    
    "service": "SecretStore",
    "serviceDefinitionId": "0",
    "serviceEndpoint": "http://secretstore.org:12001"
  }
```

All the urls in this scenario are encrypted at once. It means if a DDO has multiple URLs, an array in JSON format will be created with all the URLs.
This array will be encrypted and the HASH returned will be added as one entry of the files attribute. Example:

A DDO with 3 urls as input:
```json
   "files": [
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
            "url": "80684089027358963495379879a543085340abffh21983ddhiiee982143827abcc2"
          }
   ]
```

In this case, the following text will be encrypted:
```
[{"url":"234ab87234acbd09543085340abffh21983ddhiiee982143827423421","checksum":"efb2c764274b745f5fc37f97c6b0e761","contentLength":"4535431","resourceId":"access-log2018-02-13-15-17-29-18386C502CAEA932" }, { "url":"234ab87234acbd6894237582309543085340abffh21983ddhiiee982143827423421","checksum":"085340abffh21495345af97c6b0e761","contentLength":"12324"},{"url":"80684089027358963495379879a543085340abffh21983ddhiiee982143827abcc2"}]
```

After the encryption, the previous URLs will be removed and the encrypted HASH added to the DDO.
```json
 "encryptedFiles": "ihfuewufhwieuhcciweuhiweucnksdcnksdncksdvndksjn3u34n3unnfrunf4u3"
```

More information about the integration of the Secret Store can be found [in the Dev-Ocean repository](https://github.com/oceanprotocol/dev-ocean/blob/master/doc/architecture/secret-store.md).

#### Rsa Public and Private Keys

This is the case when PUBLISHER wants to encrypt the URLs related with the contents. The PUBLISHER encrypts individually each URL using the
RSA encryption protocol according to PKCS#1 OAEP. In Python can be used the [PyCrypto library](https://pythonhosted.org/pycrypto/Crypto.Cipher.PKCS1_OAEP-module.html) to implement this.

Because each URL is encrypted individually, given an input of N URLs, an output of N URLs encrypted will be created in the DDO.

In this case the CONSUMER doesn't need to decrypt the URLs, is the PUBLISHER who decrypt the URLs during the consumption flow.

Example:
```json
service": [{    
    "type": "Authorization",    
    "service": "RSAES-OAEP",
    "serviceDefinitionId": "0"
  }    
```


### Implementation details

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
   * BRIZO: secret store : grant access
   * SQUID: consumer: retrieve data

- SQUID: A function for cancelling payments
- SQUID: A function for publishing an Asset
- SQUID: A function for purchasing an Asset, which consists of:
   * signing the given service
   * registering service ID locally
   * calling the purchase endpoint
   * subscribing to events

- BRIZO: An endpoint for accepting purchases and instantiating service agreements
- BRIZO: Consume endpoints for providing decrypted keys and purchased data
