```
shortname: 8/ASSET-DDO
name: Assets Metadata Ontology
type: Standard
status: Raw
version: 0.2
editor: Aitor Argomaniz <aitor@oceanprotocol.com>
contributors: Kiran Karkera <kiran.karkera@dex.sg>, Enrique Ruiz <enrique@oceanprotocol.com>, Mike Anderson <mike.anderson@dex.sg>, Matthias Kretschmann <matthias@oceanprotocol.com>, Marcus Jones <marcus@oceanprotocol.com>
```

**Table of Contents**

<!--ts-->

   * [Assets Metadata Ontology](#assets-metadata-ontology)
      * [Change Process](#change-process)
      * [Language](#language)
      * [Motivation](#motivation)
      * [Life Cycle of Metadata](#life-cycle-of-metadata)      
      * [Base Attributes](#base-attributes)
      * [Curation Attributes](#curation-attributes)
      * [Additional Information](#additional-information)
      * [Example](#example)
      * [References](#references)

<!--te-->

# Assets Metadata Ontology

`version 0.2`

Every Asset (dataset, algorithm, etc.) in the Ocean Network has an associated Decentralized Identifier (DID) and DID document / DID Descriptor Object (DDO). Why? Because Assets without proper descriptive metadata have poor visibility and discoverability.

See [OEP 7/DID](../7/README.md) for information about the overall structure of Ocean DDOs and DIDs.

This OEP is about one particular part of Ocean DDOs: the Asset Metadata, a JSON object with information about the Asset.
This OEP defines the Assets Metadata Ontology, i.e. the schema for the Asset Metadata.
It's based on the public schema.org [DataSet schema](https://schema.org/Dataset).

This OEP doesn't detail the exact method of registering Assets on-chain or storing DDOs.

## Change Process

This document is governed by [OEP 2/COSS](../2/README.md).

## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.

## Motivation

The main motivations of this OEP are to:

* Specify the common attributes that MUST be included in any Asset Metadata stored in the Ocean Network
* Normalize the attributes to use in any curation process, to provide a common structure to sort and filter the DDOs
* Identify the recommended additional attributes that SHOULD be included in a DDO to facilitate Asset search
* Provide an example of an Asset Metadata object and additional links for reference

## Life Cycle of Metadata

Metadata is first created by the publisher of the asset. The publisher has knowledge of the file URL's, and they are stored in plaintext in the **files** attribute. After publication, the metadata store (Aquarius) will return the Metadata with this data encrypted. The result will be a single ciphertext of the attribute. The **dateCreated** attribute is created by the metadata store. The **curation** attribute is furthermore not created by the publisher, but by the metadata store. As such, there are 2 flavors of metadata:

1) Local metadata - Created by the publisher of the asset, added to the `DDO` and sent to the metadata store API. 

2) Remote metadata - Created and existing internally by the metadata store, and sent as a result of querying the API for discoverability of assets. 

## Base Attributes

The base attributes are recommended to be included in the Asset Metadata.
A part of the most basic, Ocean Protocol doesn't require attributes, it's up to the PROVIDERS to use the optional attributes.
The stored _values_ can be empty. The following are the base attributes:

Some attributes are required by only the metadata store *(remote)* and others are mandatory for *(local)* metadata only. If required or not by both, they are marked with *Yes/No* in the *Required* column.

Attribute       |   Type        |   Required    | Description
----------------|---------------|---------------|----------------------
**name**        | Text          | Yes           | Descriptive name or title of the Asset.
**dateCreated** | DateTime      | Yes   | The date on which the asset was created by the originator. ISO 8601 format, Coordinated Universal Time, (`2019-01-31T08:38:32Z`).
**author**      | Text          | Yes           | Name of the entity generating this data (e.g. Tfl, Disney Corp, etc.).
**license**     | Text          | Yes           | Short name referencing the license of the asset (e.g. Public Domain, CC-0, CC-BY, No License Specified, etc. ). If it's not specified, the following value will be added: "No License Specified".
**price**       | Number        | Yes           | Price of the asset. If not specified, then the default is 0.
**files**       | Array of files object | (local)     | Array of File objects including the encrypted file urls. Further metadata about each file is stored: contentType, checksum (optional), content length in bytes (optional), encoding (optional), compression (optional) and remote resourceId (optional)
**encryptedFiles** | Text         | (remote)    | Encrypted string of the **files** attribute. 
**checksum**    | Text          | Yes           | SHA3 Hash of concatenated values : [list of all file checksums] + name + author + license + did
**categories**  | Array of Text | No            | Optional array of categories associated to the Asset.
**tags**        | Array of Text | No            | Keywords or tags used to describe this content. Multiple entries in a keyword list are typically delimited by commas. Empty by default.
**type**        | Text          | No            | Type of the Asset. Helps to filter by the type of asset. It could be for example ("dataset", "algorithm", "container", "workflow", "other"). It's up to the PROVIDER or MARKETPLACE to use a different list of types or not use it.
**description** | Text          | No            | Details of what the resource is. For a dataset, this attribute explains what the data represents and what it can be used for.
**copyrightHolder**| Text       | No            | The party holding the legal copyright. Empty by default.
**workExample** | Text          | No            | Example of the concept of this asset. This example is part of the metadata, not an external link.
**links**       | Array of Link | No            | Mapping of links for data samples, or links to find out more information. Links may be to either a URL or another Asset. We expect marketplaces to converge on agreements of typical formats for linked data: The Ocean Protocol itself does not mandate any specific formats as these requirements are likely to be domain-specific.
**inLanguage**  | Text          | No            | The language of the content. Please use one of the language codes from the [IETF BCP 47 standard](https://tools.ietf.org/html/bcp47).
**curation** | Object | (remote) | The curation attributes such as votes or rating. 
**additionalInformation** | Object | No | A catch-all for extra attributes and custom functionality. Defined by the metadata store / marketplace operator. 

## files

The `files` attribute includes the details necessary to consume and validate the data.
This attribute include an array of objects of type `file`. The type file has the following attributes:

| Attribute         | Required | Description                                         |
| ----------------- | -------- | --------------------------------------------------- |
| **url**           | yes      | Content URL. The URL is encrypted after publication. |
| **index**           | yes      | Index number starting from 0 of the file. |
| **contentType**   | no       | File format, if applicable. |
| **checksum**      | no       | Checksum of the file using your preferred format (i.e. MD5). Format specified in **checksumType**. If it's not provided can't be validated if the file was not modified after registering. |
| **checksumType**  | no       | Format of the provided checksum. Can vary according to server (i.e Amazon vs. Azure) |
| **contentLength** | no       | Size of the file in bytes.                        |
| **encoding**      | no       | File encoding (e.g. UTF-8). |
| **compression**   | no       | File compression (e.g. no, gzip, bzip2, etc). |
| **resourceId**    | no       | Remote identifier of the file in the external provider. It is typically the remote id in the cloud provider. |

## curation 

To normalize the different possible rating attributes after a curation process, this is the normalized list of curation attributes:

Attribute       |   Type           |   Required    | Description
----------------|------------------|---------------|----------------------
**rating**      | Number (decimal) | Yes           | Decimal value between 0 and 1. 0 is the default value.
**numVotes**    | Integer          | Yes           | Number of votes. 0 is the default value.
**schema**      | Text             | No            | Schema applied to calculate the rating.
**isListed**   | Boolean          | No            | Flag unsuitable content. False by default. If it's true, the content must not be returned.

## additionalInformation

These are examples of attributes that can enhance the discoverability of a resource:

| Attribute             | Description                                                                                                                  |
| -                     | -                                                                                                                            |
| **sla**               | Service Level Agreement.                                                                                                      |
| **industry**          |                                                                                                                              |
| **updateFrequency**   | An indication of update latency - i.e. How often are updates expected (seldom, annually, quarterly, etc.), or is the resource static that is never expected to get updated. |
| **termsOfService**    |                                                                                                                              |
| **privacy**           |                                                                                                                              |
| **keyword**           | A list of keywords/tags describing a dataset.                                                                                 |
| **structured-markup** | A link to machine-readable structured markup (such as ttl/json-ld/rdf) describing the dataset.                                |

The publisher of a DDO MAY add additional attributes or change the above object definition. 

## Example - Local metadata

Here is an example of an Asset metadata object following the above-described schema for a local publisher metadata file. 

```json
{
  "base": {
    "name": "10 Monkey Species Small",
    "dateCreated": "2012-02-01T10:55:11Z",    
    "author": "Mario",
    "license": "CC0: Public Domain",
    "price": 10,
    "files": [
      {
        "index":0,
        "contentType": "application/zip",
        "encoding": "UTF-8",
        "compression": "zip",
        "checksum": "2bf9d229d110d1976cdf85e9f3256c7f",
        "checksumType": "MD5",
        "contentLength": 12057507,
        "url": "https://s3.amazonaws.com/assets/training.zip"
      },
      {
        "index":1,
        "contentType": "text/txt",
        "encoding": "UTF-8",
        "compression": "none",
        "checksum": "354d19c0733c47ef3a6cce5b633116b0",
        "checksumType": "MD5",
        "contentLength": 928,
        "url": "https://s3.amazonaws.com/datacommons/monkey_labels.txt"
      },
      {
        "index":2,
        "url": "https://s3.amazonaws.com/datacommons/validation.zip"
      }
    ],
    "checksum": "",
    "categories": [
      "image"
    ],
    "tags": [
      "image data",
      "classification",
      "animals"
    ],
    "type": "dataset",
    "description": "EXAMPLE ONLY ",
    "copyrightHolder": "Unknown",
    "workExample": "image path, id, label",
    "links": [
      {
        "name": "example model",
        "url": "https://drive.google.com/open?id=1uuz50RGiAW8YxRcWeQVgQglZpyAebgSM"
      },
      {
        "name": "example code",
        "type": "example code",
        "url": "https://github.com/slothkong/CNN_classification_10_monkey_species"
      },
      {
        "url": "https://s3.amazonaws.com/datacommons/links/discovery/n5151.jpg",
        "name": "n5151.jpg",
        "type": "discovery"
      },
      {
        "url": "https://s3.amazonaws.com/datacommons/links/sample/sample.zip",
        "name": "sample.zip",
        "type": "sample"
      }
    ],
    "inLanguage": "en"
  }
}
```

## Example - Remote metadata

Similarly, this is how the metadata file would look as a response to querying Aquarius (remote metadata). Note that *files* is replaced with *encryptedFiles*, and *curation* is added.



```json
{
  "base": {
    "name": "10 Monkey Species Small",
    "dateCreated": "2012-02-01T10:55:11Z",
    "author": "Mario",
    "license": "CC0: Public Domain",
    "price": 10,
    "files": [
      {
        "index":0,
        "contentType": "application/zip",
        "encoding": "UTF-8",
        "compression": "zip",
        "checksum": "2bf9d229d110d1976cdf85e9f3256c7f",
        "checksumType": "MD5",
        "contentLength": 12057507
      },
      {
        "index":1,
        "contentType": "text/txt",
        "encoding": "UTF-8",
        "compression": "none",
        "checksum": "354d19c0733c47ef3a6cce5b633116b0",
        "checksumType": "MD5",
        "contentLength": 928
      },
      {
        "index":2
      }
    ],
    "encryptedFiles": "234ab87234acbd095430853424ab87234acbd09543085340abffh21983ddhiiee9821438274234210abffh21983ddhiiee982143827423421",
    "checksum": "",
    "categories": [
      "image"
    ],
    "tags": [
      "image data",
      " animals"
    ],
    "type": "dataset",
    "description": "EXAMPLE ONLY ",
    "copyrightHolder": "Unknown",
    "workExample": "image path, id, label",
    "links": [
      {
        "name": "example model",
        "url": "https://drive.google.com/open?id=1uuz50RGiAW8YxRcWeQVgQglZpyAebgSM"
      },
      {
        "name": "example code",
        "type": "example code",
        "url": "https://github.com/slothkong/CNN_classification_10_monkey_species"
      },
      {
        "url": "https://s3.amazonaws.com/datacommons/links/discovery/n5151.jpg",
        "name": "n5151.jpg",
        "type": "discovery"
      },
      {
        "url": "https://s3.amazonaws.com/datacommons/links/sample/sample.zip",
        "name": "sample.zip",
        "type": "sample"
      }
    ],
    "inLanguage": "en",
    "curation": {
        "rating": 0.93,
        "numVotes": 123,
        "schema": "Binary Voting",
        "isListed": true
     }
  }
}
```

## References

[Schema.org](https://schema.org/) is a collaborative, community activity with a mission to create, maintain, and promote schemas for structured data on the Internet.
Data types use the [Schema.org primitive data types](https://schema.org/DataType).

Schemas:

* DataSet - https://schema.org/Dataset
* FileSize - https://schema.org/fileSize
* Common license types for datasets - https://help.data.world/hc/en-us/articles/115006114287-Common-license-types-for-datasets
