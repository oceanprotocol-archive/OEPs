```
shortname: 11/ACL
name: On-Chain Access Control using Service Agreements
type: Standard
status: Raw
editor: Aitor Argomaniz <aitor@oceanprotocol.com>
contributors: Lev Berman <ldmberman@gmail.com>
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
         * [Consuming](#consuming)
            * [Execution of SLA](#execution-of-sla)
               * [Lock payment condition](#lock-payment-condition)
               * [Grant access condition](#grant-access-condition)
               * [Release payment condition](#release-payment-condition)
         * [Consuming the data](#consuming-the-data)
            * [Cancel payment condition](#cancel-payment-condition)
         * [Modules to be implemented](#modules-to-be-implemented)
            * [Secret Store](#secret-store)




---


# On-Chain Access Control using Service Agreements

This OEP introduce the integration pattern to apply in order to use **Service Agreements (SA)** as contract between parties interacting in a transaction.
This OEP evolves the existing OEP-10 [OEP-10](../10/README.md), using the SA as the core element to orchestrate the publish/consume transactions for multiple services.

It's out of the scope to detail the Service Agreements implementation. Service Agreements are described as part of the Dev-Ocean repository.

## Change Process

This document is governed by the [2/COSS](../2/README.md) (COSS).


## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.


## Motivation

The main motivations of this OEP are:

* Evolve the OEP-10 to support more complex contract interactions between consumers and publishers of services
* Detail the main characteristics of this interaction
* Introduce an alternative negotiation to the one described in the [OEP-10](../10/README.md)
* Specify what are the pros/cons of this approach
* Identify the modifications to apply to integrate this approach
* Identify the API methods to expose in the different libraries
* Have a more secure/stable approach


## Actors

The different actors interacting in this flow are:

* PUBLISHERS - Provide access to Assets and/or Services
* CONSUMERS - Want to get access to Assets or Services
* MARKETPLACES or TRIBES - Store the DDO/Metadata related with the Assets

### Technical components

The following technical pieces are involved in the end to end publishing and consumption flow:

* [MARKETPLACE](https://github.com/oceanprotocol/pleuston) - Exposes a web interface allowing the users to publish and purchase assets. Facilitates also the discovering capabilities of assets.
* SQUID - Library encapsulating the Ocean Protocol business logic. Interacts with all the different components/api's of the system. It's provided in the main following languages:
  - [Squid Javascript](https://github.com/oceanprotocol/squid-js) - Javascript version of Squid to be integrated in the Frontend applications.
  - [Squid Python](https://github.com/oceanprotocol/squid-py) - Python version of Squid to be integrated in the Backend applications. Main users are data scientists.
* [KEEPER CONTRACTS](https://github.com/oceanprotocol/keeper-contracts) - Provide the Service Agreement (SA) business logic
* [SECRET STORE](https://github.com/oceanprotocol/parity-ethereum) - Included as part of the Parity Ethereum client. Allows the PUBLISHER to encrypt the Asset url. Integrate the SA to authorize on-chain the decryption of the Asset url by the CONSUMER
* [BRIZO](https://github.com/oceanprotocol/brizo) - Microservice to be executed by the PUBLISHER's. It exposes the HTTP REST API allowing to access to PUBLISHER Assets or additional services like computation.
* [AQUARIUS](https://github.com/oceanprotocol/aquarius) - Microservice to be executed by the MARKETPLACES's. Allows to create, update, delete and search the Asset's metadata registered by the PUBLISHERS. This Metadata, is included as part of a [DDO](../7/README.md) including also the Services associated to the Asset (Consumption, Computation, etc.).

![Actors running Components](images/software-run-by-actors.png)

## Flow

This section describes the Asset purchase flow in detail. It should be straightforward to implement the flow by reading this description although the actual implementation may deviate from it.
The detailed description is an attempt to account for important edge cases and to create a good reference for the authors of particular implementations.

There are some parameters used in this flow:

* **did** -
* **serviceAgreementId** - Is the unique id refering to a Service Agreement establish between a PUBLISHER and a CONSUMER. The CONSUMER (Squid) is the one creating this random unique serviceId.
* **serviceDefinitionId** - Identify one service in the array of services included in the DDO. Are created by the PUBLISHER (internally in Squid) when the DDO is created and different services are associated.
* **templateId** - Identify a unique Service Agreement template. The service agreement is the instance of one existing template. Initially the following templates are supported:
  - **Access** template, where the templateId is `044852b2a670ade5407e78fb2863c51000000000000000000000000000000000`
  - **FitchainCompute** template, where the templateId is `044852b2a670ade5407e78fb2863c51000000000000000000000000000000001`
  - **CloudCompute** template, where the templateId is `044852b2a670ade5407e78fb2863c51000000000000000000000000000000002`


### Publishing

Using only one Squid call `registerAsset(asset_metadata)`, the Publisher should be able to register an Asset.

This method executes internally - everything happens off-chain.

1. PUBLISHER generates a DID. See [How to generate a DID](https://github.com/oceanprotocol/OEPs/tree/master/7#length-of-a-did). DID is a UUID in Trilobite. Might be computed as a DDO hash later.
1. PUBLISHER encrypts the URL using the Secret Store identified by DID.
1. PUBLISHER creates a DDO including the following information:
   - DID
   - Metadata
     Contains Asset name, description, etc. For details see [OEP-8](../8/README.md).
   - Public key of the Publisher
   - Encrypted URL; this URL when decrypted is not used by Consumer directly but as an identifier to request data from Publisher
   - A list of services (Access, Compute, etc)

   Each service in the list contains certain information depending on its type. Here we document two types of services required for purchasing and consuming an Asset. 

   A service of type "Access" contains:
   - Service Definition ID (`serviceDefinitionId`); helps PUBLISHER find the service definition of a DDO signed by CONSUMER
   - Service Agreement Template ID (`templateId`); has to be whitelisted, can be hardcoded in Trilobite; points to a deployed service agreement on-chain contract
   - Service endpoint (`serviceEndpoint`); Consumer's signing this service send their signatures to this endpoint
   - A list of conditions keys; condition key consists of:
     * controller contract address (included in the SLA template, repeated here for information purposes)
     * controller contract function fingerprint

     Condition keys come together with the SLA template ID, can be hardcoded in Trilobite
   - For each condition, a list of its parameter values, a timeout, and a mapping of events emitted by it to the off-chain handlers of these events
   - Each event is identified by name. Each event handler is a function from a whitelisted module

   A service of type "Access" contains 2 different endpoints:
   - **serviceEndpoint** - A URL to fetch data decryption keys from
   - **purchaseEndpoint** - A URL to initialize the service agreement

    An example of a complete DDO can be found [here](ddo.example.json).

    ```
    {
    "@context": "https://w3id.org/future-method/v1",
    "id": "did:op:08a429b8529856d59867503f8056903a680935a76950bb9649785cc97869a43d",
    "publicKey": [{ .. }],
    "authentication": [{ .. }],
    "proof": { .. },

      "service": [{
        "type": "Access",
        "serviceDefinitionId": "0",
        "serviceEndpoint": "http://mybrizo.org/api/v1/brizo/services/access/initialize?pubKey=${pubKey}&serviceId={serviceId}&url={url}",
        "purchaseEndpoint": "http://mybrizo.org/api/v1/brizo/services/access/purchase?",
        "templateId": "044852b2a670ade5407e78fb2863c51000000000000000000000000000000000",
        "conditions": [{
                  "conditionKey': {
                    "contractAddress": "0x...",
                    "fingerprint": "0x...",
                    "functionName": "lockPayment" # needed for event handlers
                  },
                  "timeout": 10,
                  "parameters": [{
                    "price": 10
                  },
                  // A generic event listener:
                  // - listens for events with the particular service identifier
                  // - takes the event payload and passes it to the corresponding function
                  // - passes the service definition into every event handler
                  // - resolves the handler specification into a callback function
                  "events": {
                    "PaymentLocked": {
                      "actorType": ["publisher"], // or "consumer"
                      "handlers": [{
                        "moduleName": "secret_store",
                        "functionName": "grant_acess",
                        "version": "0.1"
                      }
                    },
                    ... // other event handlers
                  }
                },
                ... // other conditions
                ],
              },
              ... // other services
              ]

      }, {
        "type": "CloudCompute",
        "serviceDefinitionId": "1",
        "serviceEndpoint": "http://mybrizo.org/api/v1/brizo/services/compute?pubKey=${pubKey}&serviceId={serviceId}&algo={algo}&container={container}"
        "templateId": "044852b2a670ade5407e78fb2863c51000000000000000000000000000000002"

      }, {
        "type": "Metadata",
        "serviceDefinitionId": "2",
        "serviceEndpoint": "http://myaquarius.org/api/v1/provider/assets/metadata/{did}",
        "metadata": { }
        }]
    }


    ```
1. PUBLISHER publishes the DDO in the Metadata Store (OceanDB) using AQUARIUS.

1. PUBLISHER exposes service endpoints using BRIZO.

![Publishing Flow](images/publishing-flow.png)


### Consuming

Using Squid calls a CONSUMER can discover Assets, purchase and get access.


Squid Steps:

1. The CONSUMER uses the search method to find relevant Assets related with his query. It returns a list of DDO's.
   `assets = search("weather Germany 2017")`

1. Consumer chooses a service inside a DDO (the consumer selects a `serviceDefinitionId`).

1. The Service Agreement needs to have associated an unique `serviceAgreementId` that can be generated/provided by the CONSUMER.
In the Smart Contracts, this `serviceAgreementId` will be stored as a `bytes32`. This `serviceAgreementId` is random and is represented by a 64-character hex string (using the characters 0-9 and a-f).
The CONSUMER can generates the `serviceAgreementId` using any kind of implementation providing enough randomness to generate this id (64-characters hex string).

1. The CONSUMER signs the service details. There is a particular way of building the signature documented elsewhere.
The signature contains `(serviceAgreementId, templateId, conditionKeys, timeouts, conditionParameters)`. `serviceAgreementId` is provided by CONSUMER and has to be globally unique.
It is used to correlate events and to prevent Publisher from instantiating multiple service agreements from a single request.

1. Consumer sends `(did, serviceAgreementId, serviceDefinitionId, signature, consumerPublicKey`) to the service endpoint (BRIZO).
`serviceDefinitionId` tells PUBLISHER where to find the preimage to verify the signature. DID tells PUBLISHER which Asset to serve under these terms.

```
HTTP POST /api/v1/brizo/services/access/initialize

{
 "did": "did:op:08a429b8529856d59867503f8056903a680935a76950bb9649785cc97869a43d",
 "serviceAgreementId": "bb23s87856d59867503f80a690357406857698570b964ac8dcc9d86da4ada010",
 "serviceDefinitionId": "0",
 "signature": "cade376598342cdae231321a0097876aeda656a567a67c6767fd8710129a9dc1",
 "consumerPublicKey": "0x00a329c0648769A73afAc7F9381E08FB43dBEA72"
}

```

The execution of this endpoint should return a `HTTP 201` if everything goes okay.

   - When BRIZO receives signature from the service endpoint and verifies the signature.

   - Having the `did`, BRIZO fetch the DDO related with this `did`

   - BRIZO records `serviceAgreementId` as corresponding to the given `did`.

   - BRIZO executes the SLA by calling `ServiceAgreement.executeAgreement`, providing it with `serviceId`, `conditionKeys`, `conditionParameters`, and `timeouts`.

   - BRIZO starts listening for the `publisher` events from the events section of the service definition.

1. After receive the HTTP response confirmation from BRIZO, the CONSUMER starts listening for `consumer` events specified in the corresponding service definition,
filtering them by `serviceAgreementId`.

#### Execution of SLA

Consider an Asset purchase example. CONSUMER locks the payment. Then PUBLISHER grants access to the document. Then payment is released. CONSUMER may decrypt the document.

In general, there is a broad range of conditions which can be implemented an integrated into the described workflow.

##### Lock payment condition

Consider a sample of a service definition.

```
'ExecuteCondition': {
   'actorType': ['consumer'],
   'handlers': [{
     'module_name': 'payment',
     'function_name': 'lock_payment',
     'version': '0.1'
   },

"PaymentLocked": {
    "actorType": ["publisher"],
    "handlers": [{
        "moduleName": "accessControl",
        "functionName": "grantAccess",
        "version": "0.1"
        }]
    }
}
```

According to it, the CONSUMER listens for `ExecuteCondition` event, filters it by `serviceAgreementId`. The corresponding module with the event handler needs to be implemented in Squid.
At the same time, the PUBLISHER listens for `PaymentLocked` event filtered by `serviceAgreementId` to confirm the payment was made.

`payment.py`

```
def lock_payment(service_agreement_id, service_definition_id, price):
    condition = get_condition(service_definition_id, 'lockPayment')
    web3.call(condition['condition_key'], condition['fingerprint'], price)
```

It emits `PaymentLocked` and thus triggers the next condition. 

##### Grant access condition

```
'PaymentLocked': {
   'actor_type': ['publisher'],
   'handlers': [{
     'moduleName': 'secret_store',
     'functionName': 'grant_acccess',
     'version': '0.1'
   },
```

PUBLISHER (BRIZO) listens for `PaymentLocked` event, filters it by `serviceAgreementId`. The corresponding module with the event handler needs to be implemented in Brizo.

`secret_store.py`

```
def grant_access(service__agreement_id, service_definition_id, consumer_public_key):
    public_key = get_public_key_by_service_id(service__agreement_id)
    did = get_did(service__agreement_id)
    condition = get_condition(service_definition_id, 'grantAccess')
    web3.call(condition['condition_key'], condition['fingerpint'], public_key, did)
```

##### Release payment condition

```
'AccessGranted': {
   'actor_type': ['publisher'],
   'handlers': [{
     'moduleName': 'secret_store',
     'functionName': 'grant_acccess',
     'version': '0.1'
   },
```

PUBLISHER (BRIZO) listens for `AccessGranted` event to transfer tokens to Publisher's account.

`payment.py`

```
def release_payment(service_agreement_id, service_definition_id, price):
    condition = get_condition(service_definition_id, 'releasePayment')
    web3.call(condition['condition_key'], condition['fingerprint'], price)
```

`Release payment`, being the last condition of the agreement, finalises it and emits `AgreementFulfilled`.

### Consuming the data

```
'AccessGranted': {
   'actor_type': ['consumer'],
   'handlers': [{
     'moduleName': 'consumer',
     'functionName': 'retrieve_data',
     'version': '0.1'
   },
```

CONSUMER (Squid) listens for `AccessGranted` event to access the document.

The following are steps that have to be performed by CONSUMER to receive the data.

1. CONSUMER decrypts the URL using SQUID. This only requires the encryptedUrl existing in the DDO and the DID. A remote Parity EVM client and Secret Store cluster can be used for that.

1. CONSUMER retrieves data by calling the dedicated Brizo endpoint providing it with Consumer public key, service ID, and decrypted URL.

The consume URL may look like:

```
HTTP GET /api/v1/brizo/services/access/consume?pubKey=${pubKey}&serviceAgreementId={serviceAgreementId}&url={url}`
```

This method will return a HTTP 200 status code if everything was okay and the URL to get access to the data.

When Consumer requests purchased data, BRIZO gets 3 parameters:

* Consumer public key: `pubKey`
* Service Agreement ID: `serviceAgreementId`
* Decrypted URL: `url`. This URL is only valid if BRIZO acts as a proxy. Consumer can't download using the URL if it's not through Brizo.

Using those parameters, BRIZO does the following things:


* Find the `did` by the given `serviceAgreementId`

* Verify the given service is allowed to be consumed by the given `pubKey` and `did` using the `checkPermissions` method of the `SLA` Smart Contract.

* If CONSUMER has permissions to consume, download and provide data for the given DID

#### Cancel payment condition

Every condition can be forced to be fulfilled by one of the parties after a configured timeout, even when its dependencies are not fulfilled. It allows Consumer to cancel the payment after locking it but not receiving access to the Asset for a long period of time. Mechanics implemented in the service agreement contract ensure there are no race conditions.

Squid has to contain a function like the following and offer a convenient way to call it (CLI, UI).

```
def cancel_payment(service_agreement_id, service_definition_id):
    condition = get_condition(service_definition_id, 'cancelPayment')
    web3.call(condition['condition_key'], condition['fingerprint'])
```

### Modules to be implemented

#### Secret Store

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