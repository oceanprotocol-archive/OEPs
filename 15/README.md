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

This OEP describes the API by which Meta Agents in the Ocean ecosystem can store and provide verifiable metadata to Ocean clients.

Metadata is identified by the hash of its content, i.e. a Meta Agent acts as content-addressable storage of metadata records.

This service is primary used for storing Asset meta data, but should be open enough to provide meta data storage for other forms of metadata within the Ocean ecosystem.

## Change Process

This document is governed by the [2/COSS](../2/README.md) (COSS).

## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.


## Motivation

The main motivations of this OEP are:

* Ensure that clients can access metadata for any asset or off-chain data structure in the Ocean ecosystem
* Ensure that the data can be verifiable and persistent within the storage system.


#### Base URL

/api/v1/meta

| Name             | Method | URI                          |
|------------------|--------|------------------------------|
| addMetadata      | PUT    | /data/{metadata_id}          |
| getMetadata      | GET    | /data/{metadata_id}          |
| queryMetadata    | GET    | /query                       |


-------------------------------------------------------------------------------
### addMetaData

Add meta data to the server storage. The message body will be hashed using keccak256 and compared to the {metadata_id} to
ensure integrity.

If the metadata hash is correct, the Meta Storage Server should persist the metadata in its own storage, associating the {metadata_id} with this metadata.

Meta agents are recommended to limit usage of this service to authorised clients to avoid issues with spam.



#### Inputs
The Meta data Id is provided in the URL.

The metadata JSON is provided in the message body with a content type of `application/json`

#### Fields

```json

  < Metadata as per OEP8 >

```
#### Outputs

Should return HTTP 201 Created if the asset was successfully stored as new metadata
Should return HTTP 200 OK if the meta data was already stored


-------------------------------------------------------------------------------
### getMetaData
Get a meta data stored by the server.

#### Inputs

The metadata ID (i.e. the keccak256 hash of the metadata) is provided in the URL.

#### Outputs

The API should return the metadata of the requested metadata ID if available, or HTTP 404 if not found.

```json

  < Metadata as per OEP8 >

```

Clients can verify the integrity of the metadata by computing the keccak256 hash of the metadata JSON and comparing to
the requested metadata ID.

-------------------------------------------------------------------------------
### queryMetadata
Query a list of meta data records stored on the Server

#### Inputs

Query JSON record with the following schema:

```
schema:
  type: object
  properties:
    query:
      type: string
      description: Query to realise
      example: {"value":1}
    text:
      type: string
      description: Word to search in the document
      example: Office
    sort:
      type: object
      description: key or list of keys to sort the result
      example: {"value":1}
    offset:
      type: int
      description: Start record count
      example: 100
    count:
      type: int
      description: number of rows to return, from offset
      example: 0
```

#### Outputs
The API should return a JSON list of metadata records. The list can be limited to a maximum length to reduce bandwidth/data charges and performance issues.

```
schema:
  type: object
  properties:
    info:
      type: object
      properties:
        count:
          type:integer
          description: number of records returned
        offset:
          type: integer
          description: offset of the first record
        maxCount:
          type: integer
          description: total number of records in the query
    rows:
      type: list
      description: list of metadata
```

```json
{
  "info": {
    "count": 500,
    "offset": 100,
    "maxCount": 15345002,
  },
  "rows":  [
    < Metadata as per OEP8 >
    < Metadata as per OEP8 >
    ...
  ]
}
```
