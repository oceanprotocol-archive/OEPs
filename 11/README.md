```
shortname: 11/ACL
name: On-Chain Access Control using Service Agreements
type: Standard
status: Raw
editor: Aitor Argomaniz <aitor@oceanprotocol.com>
contributors:
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

### Publishing

Using only one SQUID call `registerAsset(asset_metadata)`, the PUBLISHER should be able to register an Asset.

This method executes internally - everything happens off-chain.

1. Publisher generates a DID. DID is a UUID in Trilobite. Might be computed as a DDO hash later.
1. Publisher encrypts the URL and publishes it in Secret Store identified by DID.
1. Publisher creates a DDO including the following information:
   - DID
   - Metadata
     Contains asset name, description, etc. For details see [OEP-8](../8/README.md).
   - Public key of the Publisher
   - A list of services

   Each service in the list contains the following information:

   - Service Definition ID; uniquely identifies this particular service within this DID
   - Service Agreement Template ID; has to be whitelisted, can be hardcoded in Trilobite; points to a deployed service agreement on-chain contract
   - Service endpoint; consumers signing this service send their signatures to this endpoint
   - A list of conditions keys; condition key consists of:
     * controller contract address (included in the SLA template, repeated here for information purposes)
     * controller contract function fingerprint

     Condition keys come together with the SLA template ID, can be hardcoded in Trilobite.
   - For each condition, a list of its parameter values and a timeout.
   - A mapping of on-chain events to off-chain event handlers. Each event is identified by name. Each event handler is a functions from a whitelisted module.

    An example:

    ```
    did = 'ocn:...'
    metadata = {
      'name': 'O',
      'description': '...'
    },
    public_key = '...'
    services = [{
      'service_definition_id': '...',
      'template_id': '...',
      'service_enddpoint': '...',
      'conditions': [{
          'condition_key': {
             'contract_address': '0x...',
             'fingerprint': '0x...',
             'function_name': 'lockPayment' # optional, can be needed for event handlers
          },
          'timeout': 10,
          'parameters': [{
              'price': 10
          },
          ...
          ],
      ...
      ],
      # A generic event listener:
      # - listens for events with the particular service identifier
      # - takes the event payload and pass it to the corresponding function
      # - passes a service definition into every event handler
      'events': {
        'PaymentLocked': {
          'actor_type': ['publisher'], # or 'consumer'
          'handlers': [{
            'module_name': 'secret_store',
            'function_name': 'grant_acess'
          },
          ...]
        }
      }
    },
    ...
    ]

    ```
1. Publisher publishes the DDO in the Metadata Store (OceanDB) using Aquarius.

1. Publisher exposes service endpoints.

### Consuming

Using Squid calls the CONSUMER can discover Assets, purchase and access

Squid Steps:

1. The user uses the search method to find relevant Assets related with his query. It returs a list of DDO's.
   `array[DDO] list = search("weather Germany 2017")`

2. Consumer chooses a service inside a DDO.

3. The consumer signs the service details. There is a particular way of building the signature documented elsewhere. The signature contains `(template ID, condition keys, timeouts, condition parameters)`.

4. Consumer sends `(did, service_definition_id, the signature, consumer public key`) to the service endpoint.

5. Consumer starts listening for `ExecuteAgreement` event, filtering by its public key, service definition ID, and DID.

6. Consumer extracts service ID from `ExecuteAgreement` event payload and starts listening for `consumer` events specified in the corresponding service definition.

7. In the meantime, Publisher receives signature from the service endpoint and verifies the signature.

8. Publisher executes the SLA by calling `ServiceAgreement.executeAgreement`. It generates service ID.

9. Publisher starts listening for the `publisher` events from the events section of the service definition.

#### Execution of SLA

Consider an asset purchase example. Consumer locks the payment. Only then Publisher grants access to the document. Only then payment is released. Consumer may decrypt the document.

##### Lock payment condition

Consider a sample of the service definition:

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
     'module_name': 'secret_store',
     'function_name': 'decrypt_document'
   },
```

1. The Consumer is listening to the `AccessGranted` event.

1. The Consumer decrypts the URL using the Secret Store.

   ```
   def decrypt_document(service_id):
       did = get_did(service_id)
       encrypted_url = get_encrypted_url(did)
       url = decrypt_document(did, encrypted_url)
       store_url(url)
   ```

1. Consumer calls Brizo (Publisher agent) to download the content related with the URL.

   ```
   HTTP GET http://mybrizo.org/api/v1/brizo/services/consume?pubKey=${pubKey}&serviceId={serviceId}&url={url}`
   ```

### Brizo (Publisher Agent)

When a user call the Consume Method /consume , Brizo get 3 parameters:

* Consumer Public Key (pubKey)
* Service Agreement Id (serviceId)
* Decrypted URL (url). This URL is only valid if Brizo acts as Proxy. User can't download using the URL if it's not through Brizo

Using those parameters, Brizo do the following things:

* Verify the Service Agreement can be consumed
  `canConsume= ServiceAgreement.checkPermissions(DDO.secretStoreKeyId, publicKey)`

* If user can consume the serviceId, Brizo get the URL given as parameter and allows the download of the Asset


### Modules to be implemented


#### Secret Store

- An utility to compute consumer signatures.

    ```
    signature = sign(service)
    ```

- A generic event handler which:
   * listens for the given web3 events
   * filters them by the given service ID
   * calls provided event handlers, passing them event payload and given service definition

- Event handlers:
   * payments : lock payment
   * payment : release payment
   * secret store : grant access
   * secret store : decrypt document

- A function for publishing an asset

- A function for purchasing an asset

---
OLD


6. Call the execution

Sign all the conditions and send to the Publisher templateId, ConditionKeys, Value Hashes, publicKey

3. The Publisher get the Request and call the executeAgreement method
   ServiceAgreement.executeAgreement(DDO.Consume.templateId, signature,

2. The user select one DDO using the Metadata description, and retrieve information about the Service Agreements associated to that DDO.
   `array[ServiceAgreements] serviceAgreements= ServiceAgreements.getServiceAgreements(DDO.did)`

3. The user agree about the conditions and purchase the Service Agreement. He/She gets an order instance.
   `order= ServiceAgreement.purchase(serviceAgreement.id)`

4. The user after purchase, wants to get access to the resources (Assets) associated to this Service Agreement. He get's the Content of the Asset.
   `file= ServiceAgreement.consume(serviceId)`
   This function internally will do:
   - Get the DDO of the Asset associated to the Service Agreement (TBD: WHICH FUNCTION?)
   - Get the DDO.url encrypted and descrypt the URL: `SecretStore.decryptDocument(serviceId, DDO.url)`
   - Call Brizo (Publisher agent) to download the content related with the url: `HTTP GET http://mybrizo.org/api/v1/brizo/services/consume?pubKey=${pubKey}&serviceId={serviceId}&url={url}`

