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

Using only one SQUID call (registerAsset method), the PUBLISHER should be able to register a Service Agreement.

This method executes internally (everything happens off-chain):

1. We create a DDO including the following information:
   - Metadata
     * name, like 'Ocean Dataset of Fish'
     * examples, like a CSV sample
     * etc
   - public key of the Publisher
   - DID (is generated automatically using UUID initially, after trilobite hashing the DDO)
   - A list of services

   Each service in the list contains the following information:
   - Service Agreement Template ID; has to be whitelisted, can be hardcoded in Trilobite
   - A list of conditions keys; condition key consists of:
     * controller contract address (included in the SLA template, repeated here for information purposes)
     * controller contract function fingerprint

     Condition keys come together with the SLA template ID, can be hardcoded in Trilobite.
   - For each condition, a list of its parameter values.
   - A mapping of events to event handlers. Each event is identified by name. Each event handler is a functions from a whitelisted module.

An example:

```
did = 'ocn:...'
metadata = {
  'name': 'O',
  'description': '...'
},
public_key = '...'
services = [{
  'template_id': '...',
  'conditions': [{
      'condition_key': {
         'contract_address': '0x...',
         'fingerprint': '0x...',
         'function_name': 'lockPayment' # optional
      },
      'timeouts': [..],
      'parameters': [{
          'asset_id': 'my-shiny-dataset',
          'price': 10
      },
      ...
      ],
  ...
  ],
  # A generic event listener would take the event payload and pass it to the corresponding function.
  # A generic event listener only listens for events with the particular service identifier.
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
registerAsset(did, metadata, public_key, services)
```

2. We publish the DDO in the MetadataStore (OceanDB) using Aquarius.
   ```
   result = Metadata.publishDDO(ddo)
   ```


### Consuming

Using Squid calls the CONSUMER can discover Assets, purchase and access

Squid Steps:

1. The user uses the search method to find relevant Assets related with his query. It returs a list of DDO's.
   `array[DDO] list= search("weather Germany 2017")`

2. The consumer sends to the Publisher the Signature (templateId, conditions, timeouts, listValueHashs [prices, etc]), publicKey

3. Verify the signature, verify the price checking, recreate the value hash

4. The Publisher execute the Agreement calling the Service Agreement Smart Contract
   ServiceAgreement.executeAgreement(templateId, signature, consumerPublicKey, listValueHash[includes the secretStoreKeyId], listTimeouts)
   emit ExecuteCondition for each condition on the Agreement
   emit ExecuteAgreement(serviceId, templateId, stateAgreement, publisher, consumer (indexed))

5. Consumer should listen the ExecuteAgreement event

6. Consumer should listen after to multiple ExecuteCondition filtered by serviceId

7. For each condition event, the consumer need to execute one action (TemplateID specific)
   In the Access Use case, the action is going to lockPayment

8. The Publisher is going to listen the LockPayment event

9. The Publisher encrypts the contents and puts the decryption keys into Secret Store, `encryptedUrl = encryptDocument(serviceId, url)`

10. The Publisher is going to grantAccess to the Publisher
   `grantAccess(serviceId, consumerPublicKey, encryptedUrl)`
   emit GrantAccess containing `encryptedUrl`

11. The Consumer is listening to the GrantAccess event filreting by serviceId

12. The Consumer Decrypt the url using the secretStore
   `url= SecretStore.decryptDocument(DDO.secretStoreKeyId, DDO.url)`

13. Consumer call Brizo (Publisher agent) to download the content related with the url:
   `HTTP GET http://mybrizo.org/api/v1/brizo/services/consume?pubKey=${pubKey}&serviceId={serviceId}&url={url}`



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

1. If the Service is not about a commons Asset, we encrypt the Asset url using the secret store. We give as parameter the serviceId and the url. We get an encrypted url.
   ```
   encrypte_url = SecretStore.encrypt_document(service_id, url)
   public_key = get_public_key_by_service_id(service_id)
   grant_access(public_key, encrypted_url) // calls on-chain function, like AccessConditions.grantAccess
   ```

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

