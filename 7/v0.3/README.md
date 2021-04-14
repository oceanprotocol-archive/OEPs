# OEP-7: Decentralized Identifiers

```
shortname:      7/DID
name:           Decentralized Identifiers
type:           Standard
status:         Draft
version:        0.3
editor:         Alex Coseru <alex@oceanprotocol.com>
contributors:   Matthias Kretschmann <matthias@oceanprotocol.com>,
                Ahmed Ali <ahmed@oceanprotocol.com>
```

**Table of Contents**

- [Motivation](#motivation)
- [Specification](#specification)
- [Proposed Solution](#proposed-solution)
  - [Decentralized IDs (DIDs)](#decentralized-ids-dids)
  - [DID Documents (DDOs)](#did-documents-ddos)
    - [DDO Services](#ddo-services)
  - [Integrity](#integrity)
    - [How to compute the integrity checksum](#how-to-compute-the-integrity-checksum)
    - [DID Document Proof](#did-document-proof)
    - [Length of a DID](#length-of-a-did)
    - [How to compute a DID](#how-to-compute-a-did)
- [References](#references)
- [Change Process](#change-process)
- [Language](#language)

---

This specification is based on:

* the [W3C DID specification](https://w3c-ccg.github.io/did-spec/), which was at version 0.11 as of August 2018,
* the [Ocean Protocol technical whitepaper](https://github.com/oceanprotocol/whitepaper),
* [3/ARCH](../3/README.md), and
* [4/AGENT](../4/README.md).

## Motivation

The main motivations of this OEP are:

* Design a solution to extend the current architecture to use **Decentralized Identifiers (DIDs)** and **DID Documents (DDOs)**
* Understand how to resolve DIDs into DDOs
* Establishing the mechanism to know if the DDO associated with a DID was modified
* Defining the common mechanisms, interfaces and APIs to implemented the designed solution
* Define how Ocean assets, agents and domains can be modeled with a DID/DDO data model
* Understand how DID hubs are formed, and how they integrate a business and storage layer

## Specification

Requirements are:

* The DID resolving capabilities MUST be exposed in the client libraries, enabling to resolve a DDO directly in a totally transparent way
* ASSETS are DATA objects describing RESOURCES under control of a PUBLISHER
* PROVIDERS store the ASSET metadata off-chain
* OCEAN doesn't store ASSET contents (e.g. files)
* An ASSET is modeled in OCEAN as off-chain information stored in AQUARIUS
* ASSETS information only can be modified by OWNERS or DELEGATED USERS
* ASSETS can be resolved using a Decentralized ID (DID) 
* A DID Document (DDO) should include the ASSET metadata
* Any kind of object registered in Ocean SHOULD have a DID allowing one to uniquely identify that object in the system
* ASSET DDO (and the metadata included as part of the DDO) is associated to the ASSET information stored using a common DID
* A DID can be resolved to get access to a DDO
* The function to calculate the HASH MUST BE standard

## Proposed Solution

### Decentralized IDs (DIDs)

A DID is a unique identifier that can be resolved or de-referenced to a standard resource describing the entity (a DID Document or DDO).
If we apply this to Ocean, the DID would be the unique identifier of an object represented in Ocean (i.e. the Asset ID of an ASSET or the Actor ID of a USER).
The DDO SHOULD include the METADATA information associated with this object.
The DDO is stored off-chain in Ocean.

In Ocean, a DID is a string that looks like:

```text
did:op:0ebed8226ada17fde24b6bf2b95d27f8f05fcce09139ff5cec31f6d81a7cd2ea
```

which follows [the generic DID scheme](https://w3c-ccg.github.io/did-spec/#the-generic-did-scheme).
Details about how to compute the DID are given below.

### DID Documents (DDOs)

If a DID is the index key in a key-value pair, then the DID Document is the value to which the index key points.
The combination of a DID and its associated DID Document forms the root record for a decentralized identifier.

![DDO Content](images/ddo-content.png)

A DDO document is composed of standard DDO attributes:

* `@context`
* `id`
* `created`
* `updated`
* `publicKey`
* `authentication`
* `proof`
* `verifiableCredential`
* `dataToken`
* `service`
* `isDisabled`   - optional flag, if set, will disable asset consumption, but the asset will appear in searches. This a temporary flag, publisher can switch it amy time.
* `isRetired`   - optional flag, if set, will disable the display of the asset & consumption. This is permanent flag.

Asset metadata can be included as one of the objects inside the `"service"` array, with type `"metadata"`.

#### DDO Services

Each type of asset (dataset, algorithm, workflow, etc, ..) typically will have associated different kind of services. There are multiple type of services that are commonly added to all the assets:
* metadata - describing the asset
* provenance - describing the asset provenance
* access - describing how the asset can be downloaded
* compute - describing how the asset can be computed upon

Each service is distinguished by the `DDO.service.type` attribute.

Each service has an `attributes` section where all the information related to the service is added. As mandatory content, the attributes section will have a `main` sub-section. This one is important because it must include all the mandatory information that a service has to provide.

A part of the `attributes.main` sub-section, other optional sub-sections can be added (like: `attributes.curation` or `attributes.extra`) depending on the service type.

Each service has an `cost` and `timeout` (in seconds) section describing the cost (how much datatokens needs to be transferred) and how long the sevice can be used after payment. A timeout of 0 represents no time limit.

Example:

```json
"service": [  
  {  
    "index": 0,
    "type": "metadata",
    "serviceEndpoint": "https://service/api/v1/metadata/assets/ddo/did:op:0ebed8226ada17fde24b6bf2b95d27f8f05fcce09139ff5cec31f6d81a7cd2ea",
    "attributes": {  
      "main": {},
      "additionalInformation": {},
      "curation": {}
    }
  },
  {  
    "index": 1,
    "type": "access",
    "serviceEndpoint": "http://localhost:8030/api/v1/brizo/services/consume",
    "attributes": {  
      "main": {
        "cost":"10",
        "timeout":0
      },
      "additionalInformation": {}
    }
  },
  {  
    "index": 2,
    "type": "compute",
    "serviceEndpoint": "http://localhost:8030/api/v1/brizo/services/compute",
    "attributes": {  
      "main": {
        "cost":"10",
        "timeout":3600
      },
      "additionalInformation": {}
    }
  }
]
```

- You can find a [complete example of a DDO](ddo-example.json).
- You can find a complete reference of the asset metadata in [OEP-8](8).
- You can find a complete [real world example of a DDO](https://w3c-ccg.github.io/did-spec/#real-world-example) with extended services added, as part of the W3C DID spec.


#### DID Document Proof

Since V3, the metadata is stored on chain, so we don't need additional proofs, because we already have the transaction sender.

#### Length of a DID

The length of a DID must be compliant with the underlying storage layer and function calls. Given that decentralized virtual machines make use of contract languages such as Solidity and WASM, it is advised to fit the DID in structures such as `bytes32`.

It would be nice to store the `did:op:` prefix in those 32 bytes, but that means fewer than 32 bytes would be left for storing the rest (25 bytes since "did:op:" takes 7 bytes if using UTF-8). If the rest is a secure hash, then we need a 25-byte secure hash, but secure hashes typically have 28, 32 or more bytes, so that won't work.

Only the hash value _needs_ to be stored, not the `did:op:` prefix, because it should be clear from context that the value is an Ocean DID.

#### How to compute a DID

The DID (`id`) string begins with `did:op:` and is followed by a string representation of a bytes32.

In V3, the DID is based on the datatoken address.



## References

* [DID Spec from the W3C Credentials Community Group](https://w3c-ccg.github.io/did-spec/)
* [DID Spec from _Rebooting the Web of Trust_](https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-fall2016/blob/master/topics-and-advance-readings/did-spec-working-draft-03.md)

## Change Process

This document is governed by [OEP 2/COSS](../2/README.md).

## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.
