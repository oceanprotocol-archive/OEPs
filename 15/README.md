```
shortname: 15/META-API
name: Metadata Agent API
type: Standard
status: Raw
editor: Bill Barman <bill@oceanprotocol.com>
contributors: Mike Anderson <mike.anderson@dex.sg>
```


Table of Contents
=================

   * [Table of Contents](#table-of-contents)



# Metadata Agent API

This OEP decribes the API by which Meta Agents in the Ocean ecosystem can store and provide verifiable metadata to Ocean clients.

Metadata is identified by the hash of its content, i.e. a Meta Agent acts as content-addressable storage of metadata files.



## Change Process

This document is governed by the [2/COSS](../2/README.md) (COSS).


## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.


## Motivation

The main motivations of this OEP are:

* Ensure that clients can access metadata for any asset in the Ocean ecosystem


## Meta storage Server API


#### Base URL

/api/v1/meta

| Name          | Method | URI                         |
|---------------|--------|-----------------------------|
| addAssetMeta  | PUT    | /assets/{asset_id}          |
| getAssetMeta  | GET    | /assets/{asset_id}          |
| getAssets     | GET    | /assets                     |


-------------------------------------------------------------------------------
### addAssetMeta

Add meta data to the server storage. The message body will be hashed using keccak256 and compared to the asset ID to
ensure integrity.

If the metadata hash is correct, the Meta Agent should persist the metadata in its own stoarge, associating the asset ID 
with this metadata.

Meta agents are recommended to limit usage of this service to authorised clients to avoid issues with spam.



#### Inputs
The asset ID is provided in the URL.

The metadata JSON is provided in the message body with a content type of `application/json`

#### Fields

```json
{
     < Metadata as per OEP8 >
}
```
#### Outputs

Should return HTTP 201 Created if the asset was successfully stored as new metadata
Should return HTTP 200 OK if the asset was already stored


-------------------------------------------------------------------------------
### getAssetMeta
Get an asset meta data stored by the server.

#### Inputs

The asset ID (i.e. the keccak256 hash of the metadata) is provided in the URL.

#### Outputs

The API should return the metadata of the requested asset if available, or HTTP 404 if not found.

```json
{
    < Metadata as per OEP8 >
}
```

Clients can verify the integrity of the metadata by computing the keccak256 hash of the metadata JSON and comparing to 
the asset ID.

-------------------------------------------------------------------------------
### getAssets
Get a list of assets stored on the Server

#### Inputs

TODO: consider search limits and offset to facilitate searching and control query size

#### Outputs
The API should return a JSON list of Asset IDs

```json
[
    "ca7c8bf73e97c26e2dca88cfcfe972e4c09a9c0e0351e77125011c78454dcc65",
    "ff62253a75f63edd24e84e56d98063140a53aba027082f4011d2bc0002495f7a"
]
```