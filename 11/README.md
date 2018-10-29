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

## Flow

This section describes the Asset purchase flow in detail. It should be straightforward to implement the flow by reading this description although the actual implementation may deviate from it. The detailed description is an attempt to account for important edge cases and to create a good reference for the authors of particular implementations.

### Publishing

Using only one Squid call `registerAsset(asset_metadata)`, the Publisher should be able to register an Asset.

This method executes internally - everything happens off-chain.

1. Publisher generates a DID. DID is a UUID in Trilobite. Might be computed as a DDO hash later.
1. Publisher encrypts the URL and publishes it in Secret Store identified by DID.
1. Publisher creates a DDO including the following information:
   - DID
   - Metadata
     Contains Asset name, description, etc. For details see [OEP-8](../8/README.md).
   - Public key of the Publisher
   - Encrypted URL; this URL when decrypted is not used by Consumer directly but as an identifier to request data from Publisher
   - A list of services

   Each service in the list contains certain information depending on its type. Here we document two types of services required for purchasing and consuming an Asset. 

   A service of type "purchase" contains:
   - Service Definition ID; uniquely identifies this particular service within this DID
   - Service Agreement Template ID; has to be whitelisted, can be hardcoded in Trilobite; points to a deployed service agreement on-chain contract
   - Service endpoint; Consumer's signing this service send their signatures to this endpoint
   - A list of conditions keys; condition key consists of:
     * controller contract address (included in the SLA template, repeated here for information purposes)
     * controller contract function fingerprint

     Condition keys come together with the SLA template ID, can be hardcoded in Trilobite
   - For each condition, a list of its parameter values and a timeout
   - A mapping of on-chain events to off-chain event handlers. Each event is identified by name. Each event handler is a functions from a whitelisted module

   A service of type "consume" contains:
   - A URL to fetch decryption keys from
   - A URL to fetch data from

    An example of a DDO:

    ```
    {
      "did": "ocn:...",
      "metadata": {
        "name": "O",
        "description": "..."
      },
      "public_key": "...",
      "encrypted_url": "..."
      "services": [{
        "type": "purchase",
        "service_definition_id": "...",
        "template_id": "...",
        "service_endpoint": "...",
        "conditions": [{
          "condition_key': {
            "contract_address": "0x...",
            "fingerprint": "0x...",
            "function_name": "lockPayment" # needed for event handlers
          },
          "timeout": 10,
          "parameters": [{
            "price": 10
          },
        },
        ... // other conditions
        ],
        // A generic event listener:
        // - listens for events with the particular service identifier
        // - takes the event payload and passes it to the corresponding function
        // - passes the service definition into every event handler
        "events": {
          "PaymentLocked": {
            "actor_type": ["publisher"], // or "consumer"
            "handlers": [{
              "module_name": "secret_store"
              "function_name": "grant_acess"
            }
          },
          ... // other event handlers
        }
      }, {
        "type": "consume",
        "decryption_keys_url": "...",
        "consume_url": "..."
      },
      ... // other services
      ]
    }
    ```
1. Publisher publishes the DDO in the Metadata Store (OceanDB) using Aquarius.

1. Publisher exposes service endpoints.

### Consuming

Using Squid calls a Consumer can discover Assets, purchase and access.

Squid Steps:

1. The Consumer uses the search method to find relevant Assets related with his query. It returns a list of DDO's.
   `assets = search("weather Germany 2017")`

2. Consumer chooses a service inside a DDO.

3. The Consumer signs the service details. There is a particular way of building the signature documented elsewhere. The signature contains `(did, service_definition_id, nonce, template ID, condition keys, timeouts, condition parameters)`. `nonce` is provided by Consumer and has to be unique across all Consumer's signatures of the same data. It prevents Publisher from instantiating multiple service agreements from a single request.

4. Consumer sends `(did, service_definition_id, the signature, Consumer public key`) to the service endpoint.

5. Consumer starts listening for `ExecuteAgreement` event, filtering by its public key, service definition ID, and DID.

6. Consumer extracts service ID from `ExecuteAgreement` event payload and starts listening for `consumer` events specified in the corresponding service definition.

7. In the meantime, Publisher receives signature from the service endpoint and verifies the signature.

8. Publisher executes the SLA by calling `ServiceAgreement.executeAgreement`. It generates service ID.

9. Publisher starts listening for the `publisher` events from the events section of the service definition.

#### Execution of SLA

Consider an Asset purchase example. Consumer locks the payment. Then Publisher grants access to the document. Then payment is released. Consumer may decrypt the document.

In general, there is a broad range of conditions which can be implemented an integrated into the described workflow.

##### Discovering service ID

Consider a sample of the service definition:

```
'ExecuteAgreement': {
   'actor_type': ['consumer'],
   'handlers': [{
     'module_name': 'service_agreement',
     'function_name': 'register_service_id'
   }
```

Squid listens for `ExecuteAgreement` event, filters it by service definition ID. It registers the service ID and subscribes to the rest of the events.

##### Lock payment condition

```
'ExecuteCondition': {
   'actor_type': ['consumer'],
   'handlers': [{
     'module_name': 'payment',
     'function_name': 'lock_payment'
   },
```

According to it, Squid listens for `ExecuteCondition` event, filters it by service ID. The corresponding module with the event handler needs to be implemented in Squid.

`payment.py`

```
def lock_payment(service_id, service_definition, price):
    condition = get_condition(service_definition, 'lockPayment')
    web3.call(condition['condition_key'], condition['fingerprint'], price)
```

It emits `PaymentLocked` and thus triggers the next condition. 

##### Grant access condition

```
'PaymentLocked': {
   'actor_type': ['publisher'],
   'handlers': [{
     'module_name': 'secret_store',
     'function_name': 'grant_acccess'
   },
```

Brizo listens for `PaymentLocked` event, filters it by service ID. The corresponding module with the event handler needs to be implemented in Brizo.

`secret_store.py`

```
def grant_access(service_id, service_definition, consumer_public_key):
    public_key = get_public_key_by_service_id(service_id)
    did = get_did(service_id)
    condition = get_condition(service_definition, 'grantAccess')
    web3.call(condition['condition_key'], condition['fingerpint'], public_key, did)
```

##### Release payment condition

```
'AccessGranted': {
   'actor_type': ['publisher'],
   'handlers': [{
     'module_name': 'secret_store',
     'function_name': 'grant_acccess'
   },
```

Brizo listens for `AccessGranted` event to transfer tokens to Publisher's account. 

`payment.py`

```
def release_payment(service_id, service_definition, price):
    condition = get_condition(service_definition, 'releasePayment')
    web3.call(condition['condition_key'], condition['fingerprint'], price)
```

`Release payment`, being the last condition of the agreement, finalises it and emits `AgreementFulfilled`.

### Consuming the data

```
'AccessGranted': {
   'actor_type': ['consumer'],
   'handlers': [{
     'module_name': 'consumer',
     'function_name': 'retrieve_data'
   },
```

Squid listens for `AccessGranted` event to access the document.

The following are steps that have to be performed by Consumer to receive the data.

1. Consumer fetches the decryption keys from the corresponding Brizo endpoint. See `decryption_keys_url` in the DDO above.

1. Consumer decrypts the URL. A Parity EVM client can be used for that.

1. Consumer retrieves data by calling the dedicated Brizo endpoint providing it with Consumer public key, service ID, and decrypted URL.

The consume URL may look like:

```
HTTP GET http://mybrizo.org/api/v1/brizo/services/consume?pubKey=${pubKey}&serviceId={serviceId}&url={url}`
```

### Brizo (Publisher Agent)

When Consumer requests purchased data, Brizo gets 3 parameters:

* Consumer public key
* Service ID
* Decrypted URL. This URL is only valid if Brizo acts as a proxy. Consumer can't download using the URL if it's not through Brizo.

Using those parameters, Brizo does the following things:

* Verify the given service is allowed to be consumed by the given public key

* If Consumer has permissions to consumer, download and provide data

### Modules to be implemented

#### Secret Store

- Squid: An utility to compute Consumer signatures.

    ```
    signature = sign(service)
    ```

- Squid, Brizo: A generic event handler which:
   * listens for the given web3 events
   * filters them by the given service ID
   * calls provided event handlers, passing them event payload and given service definition

- Event handlers:
   * Squid: service_agreement : register service ID
   * Squid: payments : lock payment
   * Brizo: payment : release payment
   * Brizo: secret store : grant access
   * Squid: consumer: retrieve data

- Squid: A function for publishing an Asset

- Squid: A function for purchasing an Asset
- Brizo: An endpoint for accepting purchases and instantiating service agreements
- Brizo: Consume endpoints for providing decrypted keys and purchased data