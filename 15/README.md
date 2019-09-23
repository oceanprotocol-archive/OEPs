# OEP-15: Distributed Asset File Storage with IPFS

```
shortname: 15/IPFS
name: Distributed Asset File Storage with IPFS
type: Standard
status: Draft
editor: Matthias Kretschmann <matthias@oceanprotocol.com>
```

**Table of Contents**

- [Abstract](#abstract)
- [Motivation](#motivation)
- [Specification](#specification)
  - [Publishing](#publishing)
  - [Consuming](#consuming)
- [Implementation](#implementation)
- [IPFS Node & Gateway](#ipfs-node--gateway)
- [Change Process](#change-process)
- [Language](#language)

---

## Abstract

This document describes how the [InterPlanetary File System (IPFS)](https://ipfs.io) is integrated in Ocean Protocol as a means to store and share public asset files in a distributed file system.

## Motivation

Storing public asset files in a centralized service poses multiple problems:

- one entity controls the data
- one entity is legally responsible for all stored data
- creates a single point of failure
- if service goes offline, asset files can't be consumed
- opening up possibilities of censorship by the entity running the service, or the service itself
- if files are moved to another location within the same service, existing URLs break

Initially created to store and efficiently move scientific data sets, IPFS solves all those issues with its goal of transforming the centralized internet into a distributed peer-to-peer network.

Files are distributed among multiple nodes, eliminating the single point of failure, legal, and censorship issues. By using content-based instead of location-based addressing of files, URLs won't break if files are moved.

Every file stored on IPFS is public by default so this OEP is meant for public files as used e.g. in [Commons](../14). Further work may be done to store files encrypted on IPFS and implement a way to decrypt them in an Ocean Protocol network.

## Specification

Every component in the Ocean Protocol stack must support publishing and consuming of asset files stored in IPFS, supporting native IPFS URLs by referencing files with their Content Identifiers (CIDs).

### Publishing

Asset files stored in IPFS should be registered in an Ocean Protocol network with the `ipfs://` protocol in an asset's DDO as its [`file.url` metadata attribute](../8/v0.3/README.md#file-attributes).

To preserve asset file names, it is encouraged to add files to IPFS by wrapping them in a folder (`wrapWithDirectory: true` with [`ipfs.add`](https://github.com/ipfs/interface-js-ipfs-core/blob/master/SPEC/FILES.md#add)) and use the `ipfs://FOLDER_CID/file.pdf` URL structure to register a file in Ocean Protocol during the publish flow.

### Consuming

When an asset file is registered with the `ipfs://CID` URL structure, the `file.contentType` value must be used to add the respective file ending during consume flow and send the correct MIME type so a user's operating system knows what to do with it.

Until the `ipfs://` protocol is supported in all browsers, an IPFS gateway shall be used to guarantee consuming of asset files stored in IPFS for a wide audience.

## Implementation

The implementation mainly concerns the components responsible for consuming, as they need to handle:

- map native IPFS URL in `file.url` to any IPFS gateway
- add file ending based on `file.contentType` for `ipfs://CID` URL structure files

This must be handled on the backend side by [Brizo](https://docs.oceanprotocol.com/concepts/components/#brizo) and the [Osmosis IPFS Driver](https://github.com/oceanprotocol/osmosis-ipfs-driver).

Additionally, adding files to IPFS should be part of the publish flow in the Commons Marketplace.

## IPFS Node & Gateway

To guarantee a reliable publishing and consume process, and to give back to the Distributed Web community, the Ocean Protocol Foundation will run and donate its own public IPFS node & gateway. 

The gateway is publicly available and can be used by anyone. The node is restricted in its HTTP API, only allowing to add files by selected resources.

This node & gateway should be used for adding, distributing, and retrieving asset files within al the processes outlined above.

## Change Process

This document is governed by [1/C4](../1/README.md) and  [2/COSS](../2/README.md).

## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", 
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and 
"OPTIONAL" in this document are to be interpreted as described in 
[BCP 14](https://tools.ietf.org/html/bcp14) 
\[[RFC2119](https://tools.ietf.org/html/rfc2119)\] 
\[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, 
they appear in all capitals, as shown here.
