```
shortname: 7/DID
name: Decentralized Identifiers
type: Standard
status: Raw
editor: Aitor Argomaniz <aitor@oceanprotocol.com>
contributors: Mike Anderson <mike.anderson@dex.sg>, Dimitri De Jonghe <dimi@oceanprotocol.com>, Troy McConaghy <troy@oceanprotocol.com>
```

**Table of Contents**

<!--ts-->

   * [Decentralized Identifiers](#decentralized-identifiers)
      * [Change Process](#change-process)
      * [Language](#language)
      * [Motivation](#motivation)
      * [Specification](#specification)
      * [Proposed Solution](#proposed-solution)
         * [Decentralized ID's (DID)](#decentralized-ids-did)
         * [DID Documents (DDO)](#did-documents-ddo)
         * [Integrity](#integrity)
            * [Length of a DID](#length-of-a-did)
            * [How to compute a DID for a DDO](#how-to-compute-a-did-for-a-ddo)
               * [Computing the Proof](#computing-the-proof)
               * [Computing the SHA-3 Hash](#computing-the-sha-3-hash)
         * [Registry](#registry)
         * [Resolver](#resolver)
      * [Changes Required](#changes-required)
      * [Changes Required](#changes-required-1)
      * [References](#references)

<!--te-->

# Decentralized Identifiers

This OEP presents an approach to using the W3C DID specification within Ocean for the following purposes:
- Providing decentralised identifiers (DIDs) for Identities on Ocean
- Providing way to address Assets as resources managed under the DID of a relevant Identity
- Providing the ability to resolve a DDO (on-chain) for relevant Identities (especially service providers who need to expose API endpoints for the ecosystem
- Providing a way to cryptographically verify metadata for assets 

For the purposes for this OEP, an Identity can be any entity that wishes to register a self-sovereign DID/DDO on
the Ocean network. Currently this is anticipated to include the following:
- Any Tribe that wishes to manage metadata for Assets, which is required to offer the Meta API
- Any off-chain service that needs to provide endpoints for Ocean Agent APIs (including marketplaces, service providers etc.)
- Any other Actor with an ethereum account who wishes to provide a DDO on Ocean

A DID/DDO is **mandatory** for any Identity wishing to offer Ocean Agent APIs to the ecosystem, because
it is necessary to do this so that a client (using e.g. squid.py) can locate relevant service endpoints
in a standardised way. A DID/DDO is optional for all other cases. In particular, data publishers
and consumers need not register a DID/DDO providing they utilise a marketplace / tribe or other
service that provides the necessary APIs on their behalf.

A DID for an Identity in Ocean takes the following format:

`did:op:cd2a3d9f938e13cd947ec05abc7fe734df8dd826`

Where the hexadecimal ID is a unique ID for the Identity

TODO: consider alternative way to allocate IDs to Identities, could include:
- Psuedo-random generation
- Use of Ethereum account
- Some form of hash of the initial DDO value

An Asset with Metadata provider by an Identity can the be addressed in the following format:

`did:op:cd2a3d9f938e13cd947ec05abc7fe734df8dd826/c5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470`

Where the two hexadecimal IDs are the ID of the Provider managing the Asset and ID of the Asset respectively. The ID 
of the asset in turn is defined as the Hash (Keccak256) of the Asset Metadata.

This scheme presents the following useful properties:
- It is possible to address the same Asset provided by multiple Service Providers (e.g. an asset is listed on multple marketplaces for sale).
- It is possible to extend this scheme to sub-Assets by extending the DID path
- There is no requirement for Assets to be registered on-chain, allowing for efficient off-chain registration of Assets and associated Metadata

## Change Process

This document is governed by the [2/COSS](../2/README.md) (COSS).


## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.


## Motivation

The main motivations of this OEP are:

* Define a standard to identify Identities in Ocean using **Decentralized Identifiers (DID)** 
* Provide a *stable* identifier for such Identity that can be communicated and stored as a reference to that identity.
* Present a solution to resolve service endpoints for Identities who are service providers via **DID description objects (DDO)**
* Provide a standard to identify Assets as resources addressed via the DID of the relevant service provider
* Defining the common mechanisms, interfaces and API's to implemented the designed solution
* Enable Ocean assets, agents and tribes to be modelled with a DID/DDO data model

## Specification

Requirements are:

* The DID resolving capabilities MUST be exposed in the client libraries, enabling to resolve a DDO directly in a totally transparent way
* IDENTITIES are self-sovereign identities who wish to register a DID/DDO on Ocean
* ASSETS are DATA objects describing RESOURCES under control of a PUBLISHER
* KEEPER stores on-chain only the essential information about ASSETS
* META AGENTS store the ASSET metadata off-chain
* KEEPER doesn't store any ASSET metadata
* An ASSET is modelled in OCEAN as ASSET Metadata stored by a Meta Agent exposing a standard Meta API and asset data stored by a storage provider exposing a standard Meta API
* ASSETS have no on-chain information (unless they are referenced in Service Agreements)
* An Identity in Ocean MUST have a DID allowing users to uniquely identify that object in the system
* A **DID** can be resolved to get access to a **DDO** using an on-chain resolver
* An Actor controlling an Identity MUST be able to update the DDO for a given DID
* An Asset ID is the HASH of the Asset Metadata
* An Asset can be identified using a DID of an IDENTITY providing the metadata of the asset, by extending the DID with the Asset ID as part of the DID path
* The function to calculate the HASH MUST BE an Ocean standard (keccak256 proposed)


## Proposed Solution

### Decentralized IDs (DIDs)

A DID is a unique identifier that can be resolved or de-referenced to a standard resource describing the entity (a DID Document or DDO).
If we apply this to Ocean, the DID would be the unique identifier of Identity represented in Ocean.

In Ocean, a DID is a string that looks like:

```text
did:op:9050a77f4acffd30a2e97bfa5abfcb9256f6f9cde60091bf9d573c534052d9fd
```

which follows [the generic DID scheme](https://w3c-ccg.github.io/did-spec/#the-generic-did-scheme).
Details about how to compute the DID are given below.

As per section 3.3 of the DID spec (https://w3c-ccg.github.io/did-spec/#paths): "A DID path SHOULD be used to address resources available via a DID service endpoint" therefore we use DID paths to address Assets managed by the relevant Actor.

As per section 9.1 of the DID spec there is no upper limit on DID length, but in Ocean we elect to limit
DIDs (including DID paths) to 2000 characters to ensure maximum interoperability with clients / URLs.

The complete specs can be found in the [W3C Decentralized Identifiers (DIDs) document](https://w3c-ccg.github.io/did-spec/)

### DID Documents (DDO)

If a DID is the index key in a key-value pair, then the DID Document is the value to which the index key points.
The combination of a DID and its associated DID Document forms the root record for a decentralized identifier.

![DDO Content](images/ddo-content.png)

A DDO document is composed of standard DDO attributes like:

* "@context"
* "id"
* "service"

Example:

```json
{
  "@context": "https://example.org/example-method/v1",
  "id": "did:op:9050a77f4acffd30a2e97bfa5abfcb9256f6f9cde60091bf9d573c534052d9fd",
  "authentication": [{ ... }],
  "service": [{
    "type": "AssetsMetadataService",
    "serviceEndpoint": "https://myservice.org/assets/"
  }]
}
```

Also it's possible to find a complete [real example of a DDO](https://w3c-ccg.github.io/did-spec/#real-world-example) with extended services added, as part of the W3C DID spec.


### Integrity

The Integrity policy for identity and metadata is a sub-specification for the Ocean Protocol allowing to validate the integrity of the Metadata associated to an on-chain object (initially an ACTOR).

Note: The first version of this spec DOES NOT include an integrity-checking mechanism. That is deferred to future versions.

#### Length of a DID

The length of a DID must be compliant with the underlying storage layer and function calls.
Given that decentralized virtual machines make use of contract languages such as Solidity and WASM, it is advised to fit the DID in structures such as `bytes32`.

It would be nice to store the "did:op:" prefix in those 32 bytes, but that means fewer than 32 bytes would be left for storing the rest (25 bytes since "did:op:" takes 7 bytes if using UTF-8). If the rest is a secure hash, then we need a 25-byte secure hash, but secure hashes typically have 28, 32 or more bytes, so that won't work.

Only the hash value _needs_ to be stored, not the "did:op:" prefix, because it should be clear from context that the value is an Ocean DID.

#### How to compute a DID

The DID ("id") string begins with "did:op:" and is followed by a string representation of a bytes32.

In the first version of this spec (to be implemented in the Trilobite release), the bytes32 part is _random_ and is represented by a 64-character hex string (using the characters 0-9 and a-f).
One way to compute such a DID is by concatenating two random UUIDs. (Each UUID is 128 bits = 16 bytes, which can be represented by a 32-character hex string with all hyphens "-" removed.)

One way NOT to compute such a DID is `sha3_256_hash(UUID).to_hex_string()`, because the space of UUIDs (16 bytes) is smaller than the space of bytes32 (32 bytes).

In the future, this spec might allow for other ways to compute a DID (e.g. the SHA3-256 hash of the DDO-without-DID).

Note: The bytes32 (a sequence of bytes) is what gets stored in a blockchain, not the final DID ("id") value. That is, the "did:op:" part doesn't have to be stored in a blockchain because it should be clear from context that the stored bytes32 is part of an Ocean DID.

### Registry

To register the different kind of objects can be stored in a **simple** register contract named **IdRegistry**.
The key of the Identity entity in Ocean is the **DID**. Associated to this DID we have a Mapping of key-value attributes,
allowing to associate publicly information to DID's. This could be used to add public information allowing for example
to discover/resolve a DID.

There are two main options to implement this:

* Associate to the DID a mapping of key-value attributes to be stored as new entries of a smart contract variable
* Emit events associated to the DID. Events works pretty well as a kind of cost effective storage. This is the recommended approach.

As a result of considering these two option, Ocean implements the Registry events keyed to the DID

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
registerAttribute("did:op:21tDAKCERh95uGgKbJNHYp", "did:op:328aabb94534935864312", "service-ddo", "https://myprovider.example.com/ddo")
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
var event = contractInstance.DidAttributeRegistered( {did: "did:op:21tDAKCERh95uGgKbJNHYp", "key": "service-ddo"}, {fromBlock: 0, toBlock: 'latest'});
```

Here in Python using web3.py:

```python
event = mycontract.events.DidAttributeRegistered.createFilter(fromBlock='latest', argument_filters={'did': 'did:op:21tDAKCERh95uGgKbJNHYp', 'key': 'service-ddo'})
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

An ASSET in the system is composed by:
- On-chain information maintained by the KEEPER (e.g. references to the Asset in Service Agreements)
- Off-chain Asset Metadata stored in an appropriate Meta Agent, which is Hashed to provide an Asset ID
- Any resources related to the provision of the asset (storage , compute etc.) maintained by other providers

Clients should validate the Asset Metadata using the Asset ID to ensure integrity of the Asset Metadata

### Proposed solution

![Sequence Diagram](images/ddo-integrity-sequence.png)
TODO: update sequence diagram

The solution included in the above diagram includes the following steps:

1. The PUBLISHER, before publishing any ASSET information, calculate the **HASH** using the **Asset Metadata** as input.
   To do that, the PUBLISHER will use from the client side a common Ocean library using the same algorithm.
1. The PUBLISHER transmits the Hash and the Asset Metadata to an appropriate Meta Agent
1. Clients may now obtain the Asset Metadata using the DID of the Meta Agent extended with the Hash of the Asset Metadata


## Changes Required

The list of changes to apply in the proposed solution are:

* Define a one-way algorithm to use to calculate the HASH function (keccak256 is suggested)
* Create a new method to calculate the HASH - CLIENT LIBRARIES
* Modify OceanMarketplace allowing to specify the HASH during the ASSET registry - KEEPER
* Integrate the HASH function with the ASSET registry process - CLIENT LIBRARIES
* Integrate the HASH calculation in the CONSUMER side - CLIENT LIBRARIES

## References

* https://w3c-ccg.github.io/did-spec/
* https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-fall2016/blob/master/topics-and-advance-readings/did-spec-working-draft-03.md
