```
shortname: 7/DID
name: Decentralized Identifiers
type: Standard
status: Raw
editor: Aitor Argomaniz <aitor@oceanprotocol.com>
contributors: 
```

<!--ts-->

Table of Contents
=================

   * [Table of Contents](#table-of-contents)
   * [Decentralized Identifiers](#decentralized-identifiers)
      * [Change Process](#change-process)
      * [Language](#language)
      * [Motivation](#motivation)
      * [Specification](#specification)
      * [Proposed Solution](#proposed-solution)
         * [Decentralized ID's (DID)](#decentralized-ids-did)
         * [Registry](#registry)
         * [Resolver](#resolver)
      * [Changes Required](#changes-required)
      * [Metadata Integrity](#metadata-integrity)
         * [Proposed solution](#proposed-solution-1)
      * [Changes Required](#changes-required-1)


<!--te-->


# Decentralized Identifiers

This OEP doesn't detail the exact method of registering ASSETS on-chain or publishing metadata in a metadata store.

This specification is based on [Ocean Protocol technical whitepaper](https://github.com/oceanprotocol/whitepaper), [3/ARCH](../3/README.md) and [4/AGENT](../4/README.md).


## Change Process

This document is governed by the [2/COSS](../2/README.md) (COSS).


## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.


## Motivation

The main motivations of this OEP are:

* Design a solution to extend the current architecture to use **Decentralized Identifiers (DID)** and **DID Documents (DDO)**
* Understand how to register information on-chain with off-chain integrity associated
* Understand how to resolve DID's into DDO's
* Design a solution facilitating to have the on-chain and off-chain information aligned
* Establishing the mechanism to know if the DDO associted to a DID was modified
* Defining the common mechanisms, interfaces and API's to implemented the designed solution
* Define how Ocean assets, agents and tribes can be modeled with a DID/DDO data model
* Understand how DID hubs are formed, and how they integrate a business and storage layer


## Specification

Requirements are:

* The DID resolving capabilities MUST be exposed in the client libraries, enabling to resolve a DDO directly in a totally transparent way
* ASSETS are DATA objects describing RESOURCES under control of a PUBLISHER
* KEEPER stores on-chain only the essential information about ASSETS
* PROVIDERS store the ASSET metadata off-chain
* KEEPER doesn't store any ASSET metadata
* OCEAN doesn't store ASSET files contents
* An ASSET is modeled in OCEAN as on-chain information stored in the KEEPER and metadata stored in OCEANDB
* ASSETS on-chain information only can be modified by OWNERS or DELEGATED USERS
* ASSETS can be resolved using a Decentralized ID (DID) included on-chain and off-chain
* A Decentralized Distributed Object (**DDO**) should includes the ASSET metadata
* Any kind of object registered in Ocean SHOULD have a DID allowing to uniquely identify that object in the system
* ASSET DDO (and the metadata included as part of the DDO) is associated to the ASSET information stored on-chain using a common **DID**
* A **DID** can be resolved to get access to a **DDO**
* ASSET DDO's can be updated without updating the on-chain information
* ASSET information stored in the keeper will include a **checksum** attribute
* The ASSET on-chain checksum attribute, includes a one-way HASH calculated using the DDO content
* After the DDO resolving, the DDO HASH can be calculated off-chain to validate if the on-chain and off-chain information is aligned
* A HASH not matching with the checksum on-chain means the DDO was modified without the on-chain update
* The function to calculate the HASH MUST BE standard


## Proposed Solution

### Decentralized ID's (DID)

A DID is a unique identifier that can be resolved or de-referenced to a standard resource describing the entity (a DID Document or DDO).
If we apply this to Ocean, the DID would be the unique identifier of an object represented in Ocean (ie. the Asset ID of an ASSET or the Actor ID of a USER).
The DDO would be the METADATA information associated to this object that is stored off-chain on Ocean.

DID schema:

```text
did-reference      = did [ "/" did-path ] [ "#" did-fragment ]
did                = "did:" method ":" specific-idstring
method             = 1*methodchar
methodchar         = %x61-7A / DIGIT
specific-idstring  = idstring *( ":" idstring )
idstring           = 1*idchar
idchar             = ALPHA / DIGIT / "." / "-"
```

In Ocean, the DID looks:

```text
did:ocn:21tDAKCERh95uGgKbJNHYp
```

The complete specs can be found in the [W3C Decentralized Identifiers (DIDs) document](https://w3c-ccg.github.io/did-spec/)

### DID Documents (DDO)

If a DID is the index key in a key-value pair, then the DID Document is the value to which the index key points.
The combination of a DID and its associated DID Document forms the root record for a decentralized identifier.

![DDO Content](images/ddo-content.png)

A DDO document is composed by the standard DDO attributes like:

* context
* id:
* services

In addition to this, the asset metadata can be included as part of the DDO inside the service entry, using the type **AssetsMetadataService**.
Example:

```json
{
  "@context": "https://example.org/example-method/v1",
  "id": "did:example:123456789abcdefghi",
  "authentication": [{ ... }],
  "service": [{
    "type": "AssetsMetadataService",
    "serviceEndpoint": "https://myservice.org/assets/",
    "metadata": {
        "title": "my asset",
        "description": "blabla"
    }
  }]
}
```

You can find a complete reference of the asset metadata in the scope of the [OEP-8](8).
Also it's possible to find a complete [real example of a DDO](https://w3c-ccg.github.io/did-spec/#real-world-example) with extended services added, as part of the w3c did spec.


### Registry

To register the different kind of objects can be stored in a **simple** register contract named **DidRegistry**.
This DidRegistry can act as generic/flexible way to associate Resources (ie. Assets) to the public providers resolving the DDO (and Metadata included) of those resources.
The key of the Identity entity in Ocean is the **DID**. Each entity will have a unique DID.
To resolve the DDO associated to a Resource (Asset), associated to this Resource DID we have the DID of the Provider giving access to this Resource.
The Provider will have associated a mapping (key - value) of attributes. One of those can be used to related with the public service returning the DDO of a specific resource.

Applied to Assets, typically are part of a Service Agreement. The suggested approach to implement this is:

* Associate the Resource (ie. Asset DID) to the Provider DID
* Each Provider will have associated a Public URL where the provider is exposing the DDO's

Here a draft **DidRegistry** implementation:

```solidity

// This piece of code is for reference only!
// Doesn't include any validation, types could be reviewed, enums, etc

contract DidRegistry {

    struct Identity {
        address owner; // owner of the Identity
        string providerDid;
    }

    struct Provider {
        mapping (bytes32 => string) attributes;
    }

    mapping (string => Identity) identities; // list of identities, mapping: identities[did] = Identity
    mapping (bytes32 => Provider) providers; // list of providers, mapping: providers[did] = Provider

    // Attributes as events (recommended)
    event DidAttributeRegistered(
        string indexed did,
        address indexed owner,
        string indexed providerDid,
        bytes32 indexed key,
        string value,
        uint updateAt
    );

    constructor(bytes32 _did, uint256 _type) public {
    }

    function register(string _resourceDID, string _providerDID, bytes32 _key, string _value) public returns (bool) {
        // It's necessary to check if the provider was already there
        // In that case is not necessary to add a new Provider (cheaper, only first time we write info about a provider)
        providers[_providerDID][_key]= _value;

        // Associating the resourceDID to the providerDID resolving the resource DID to the DDO
        identities[_resourceDID].owner= msg.sender;
        identities[_resourceDID].owner= _providerDID;

        DidAttributeRegistered(_did, msg.sender, _providerDID, _key, _value, now);
    }
}

```

To register the provider publicly resolving the DDO associated to a DID, we will register an attribute **"service-ddo"** with the public hostname of that provider:
```
registerAttribute("did:ocn:21tDAKCERh95uGgKbJNHYp", "did:ocn:328aabb94534935864312", "service-ddo", "https://myprovider.example.com/ddo")
```


### Resolver

The resolving capabilities will be encapsulated in the Ocean Client libraries (Javascript, Python, ..), allowing to resolve a DDO directly speaking with the KEEPER.
No third-party requests or API need to be integrated. This allows to have a simple a seam-less integration from the consumer side.

This is a generic definition of what could be exposed in the client libraries from an API point of view:

```java
function DDO resolve(String did)  {
  // black magic
  return ddo;
}
```

To resolve a DID to the associated DDO, some information is stored on-chain associated to the DID. In the approach recommended in the scope of this OEP, this is stored
as an attribute associated to the ```DidAttributeRegistered``` event. Because the did and key are indexed parameters of the event, a consumer in any supported web3 language,
could filter the ```DidAttributeRegistered``` events filtering by the DID and the key named **"service-ddo"**.

A DDO pointing to a DID could be resolved hierarchically using the same mechanism.

This is an example in Javascript using web3.js:

```javascript
var event = contractInstance.DidAttributeRegistered( {did: "did:ocn:21tDAKCERh95uGgKbJNHYp", "key": "service-ddo"}, {fromBlock: 0, toBlock: 'latest'});
```

Here in Python using web3.py:

```python
event = mycontract.events.DidAttributeRegistered.createFilter(fromBlock='latest', argument_filters={'did': 'did:ocn:21tDAKCERh95uGgKbJNHYp', 'key': 'service-ddo'})
```

This logic could be encapsulated in the client libraries (**Squid**) in different languages, allowing to the client applications to get the attributes enabling to resolve the DDO associated to the DID.
Using this information a consumer can query directly to the provider able to return the DDO.

Here you have the complete flow using as example a new ASSET:

![DID Resolver](images/did-resolver.png)

Steps:

1. A PUBLISHER, using the KEEPER, register the new Resource (ie. ASSET) providing the DID and the DID of the Provider acting as Public service returning the DDO of the Resource (ASSET)
1. The KEEPER register the Resource using the Service Agreement Smart Contract and after of that register the identity using the DidRegistry Smart Contract. In this point, the attribute is raised as a new event
1. The PUBLISHER publish the DDO in the metadata-store/OCEANDB provided by PROVIDER
1. A CONSUMER (it could be a frontend application or a backend software), having a DID and using a client library (Python or Javascript) get the **service-ddo** attribute associated to the DID directly from the KEEPER
1. The CONSUMER, using the provider public url, query directly to the provider passing the DID to obtain the DDO


## Changes Required

The list of changes to apply in the proposed solution are:

* Modify the function creating the id to return a DID compliant id - KEEPER
* Create the new DidRegistry Smart Contract - KEEPER
* Integrate the call to the DidRegistry contract in the OceanMarketplace or Service Agreement contract (depending of the moment of integration of this) - KEEPER
* Implement the resolving function of a DDO given a DID - CLIENT LIBRARIES


## Metadata Integrity

The Metadata Integrity policy is a sub-specification for the Ocean Protocol allowing to validate the integrity of the Metadata associated to an on-chain object (initially an ASSET).

An ASSET in the system is composed by on-chain information maintained by the KEEPER and off-chain Metadata information (DDO) stored in OCEANDB.
Technically a user could update the DDO accessing directly to the database, modifying attributes (ie. License information, description, etc.) relevant to a previous consumption agreement with an user.
The motivation of this is to facilitate a mechanism allowing to the CONSUMER of an object, to validate if the DDO was modified after a previous agreement.

### Proposed solution

![Sequence Diagram](images/ddo-integrity-sequence.png)

The solution included in the above diagram includes the following steps:

1. The PUBLISHER, before publish any ASSET information, calculate the **HASH** using the **DDO** as input.
   To do that, the PUBLISHER will use from the client side a common Ocean library using the same algorithm.
1. The PUBLISHER, in the process of registering an ASSET on-chain specifies the **HASH** in addition of the existing parameters.
1. The KEEPER register the ASSET and associate the HASH calculated using the DDO associated to the ASSET.
1. After a CONSUMER get access to an ASSET, could store internally the HASH referencing to the DDO he purchased
1. In a posterior consumption, after resolving the DDO, the CONSUMER using the client library can calculate the HASH of the DDO just obtained
1. If the HASH obtained is not the same than the HASH associated in the original purchase, means the DDO was modified afterwards
1. This, depending of the agreed conditions, could means an exit clause of some contracts

The HASH could be an optional parameter in the registering of the ASSET. If it's not specified, means the DDO can be updated without any limitation.

## Changes Required

The list of changes to apply in the proposed solution are:

* Define a one-way algorithm to use to calculate the HASH function (SHA-3 is suggested)
* Create a new method to calculate the HASH - CLIENT LIBRARIES
* Modify OceanMarketplace allowing to specify the HASH during the ASSET registry - KEEPER
* Integrate the HASH function with the ASSET registry process - CLIENT LIBRARIES
* Integrate the HASH calculation in the CONSUMER side - CLIENT LIBRARIES

## References

* https://w3c-ccg.github.io/did-spec/
* https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-fall2016/blob/master/topics-and-advance-readings/did-spec-working-draft-03.md