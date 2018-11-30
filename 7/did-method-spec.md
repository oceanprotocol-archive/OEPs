```
name: W3C Ocean Protocol DID Method Specification
type: Standard
status: Draft
editor: Aitor Argomaniz <aitor@oceanprotocol.com>
```


# Ocean Protocol DID Method Specification

[![](https://img.shields.io/badge/Status-Draft-green.svg?style=flat-square)](#Status)

This document specifies the Ocean Protocol
[DID Method](https://w3c-ccg.github.io/did-spec/#specific-did-method-schemes) [`did:op`].

This specification conforms to the requirements specified in the DID specification currently published by the W3C
Credentials Community Group. For more information about DIDs and DID method specifications, please see
[DID Primer](https://git.io/did-primer) and [DID Specification](https://w3c-ccg.github.io/did-spec).

## Method Name

The namestring that shall identify this DID method is: `op` (acronym of Ocean Protocol).

A DID that uses this method **MUST** begin with the following prefix: `did:op:`. Per the DID specification,
this prefix MUST be in lowercase. The format of remainder of the DID, after this prefix, is specified below in
the section on [Method Specific Identifiers](#method-specific-identifiers).

## Method Specific Identifiers

Ocean Protocol DIDs conform with [the Generic DID Scheme](https://w3c-ccg.github.io/did-spec/#the-generic-did-scheme)
described in the DID spec. The format of the `idstring` is described below in
[ABNF](https://tools.ietf.org/html/rfc5234):

```
op-did               = "did:op:" idstring
idstring                =  64-character hex string
```

### Length of a DID

The length of a DID must be compliant with the underlying storage layer and function calls. In Ocean Protocol, the `idstring` will be stored as identifier as reference in Smart Contracts to reference some resources.
Given that decentralized virtual machines make use of contract languages such as Solidity and WASM, it is advised to fit the DID in structures such as `bytes32`.

It would be nice to store the "did:op:" prefix in those 32 bytes, but that means fewer than 32 bytes would be left for storing the rest (25 bytes since "did:op:" takes 7 bytes if using UTF-8). If the rest is a secure hash, then we need a 25-byte secure hash, but secure hashes typically have 28, 32 or more bytes, so that won't work.

Only the hash value _needs_ to be stored, not the "did:op:" prefix, because it should be clear from context that the value is an Ocean DID.

### How to compute a DID

The DID `op-did` string begins with `did:op:` and is followed by a string representation of 64 hex characters. It allows to store this information in a bytes32 data structure.

Any random 64-character hex string can be used to create a `idstring`
In the current implementation of Ocean, the `idstring` is computed by concatenating two random UUIDs. (Each UUID is 128 bits = 16 bytes, which can be represented by a 32-character hex string with all hyphens "-" removed.)

One way NOT to compute such a DID is `sha3_256_hash(UUID).to_hex_string()`, because the space of UUIDs (16 bytes) is smaller than the space of bytes32 (32 bytes).

Note: The bytes32 (a sequence of bytes) is what gets stored in a blockchain, not the final DID ("id") value. That is, the "did:op:" part doesn't have to be stored in a blockchain because it should be clear from context that the stored bytes32 is part of an Ocean DID.

### Example

A valid Ocean Protocol DID might be:
`did:op:0ebed8226ada17fde24b6bf2b95d27f8f05fcce09139ff5cec31f6d81a7cd2ea`

## References

* [Decentralized Identifiers (DIDs)](https://w3c-ccg.github.io/did-spec)
* [OEP7: Ocean Protocol Decentralized Identifiers](README.md)


