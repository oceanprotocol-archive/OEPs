```
shortname: 8/ASSET-DDO
name: Assets Metadata Ontology
type: Standard
status: Raw
version: 0.3
editor: Aitor Argomaniz <aitor@oceanprotocol.com>
contributors: Kiran Karkera <kiran.karkera@dex.sg>,
              Enrique Ruiz <enrique@oceanprotocol.com>,
              Mike Anderson <mike.anderson@dex.sg>,
              Matthias Kretschmann <matthias@oceanprotocol.com>,
              Marcus Jones <marcus@oceanprotocol.com>,
              Troy McConaghy <troy@oceanprotocol.com>
```

**Table of Contents**

<!--ts-->

   * [Assets Metadata Ontology](#assets-metadata-ontology)
      * [Change Process](#change-process)
      * [Language](#language)
      * [Motivation](#motivation)
      * [Life Cycle of Metadata](#life-cycle-of-metadata)
         * [Local Metadata](#local-metadata)
         * [Remote Metadata](#remote-metadata)
      * [Metadata Attributes](#metadata-attributes)
         * [Base Attributes](#base-attributes)
            * [File Attributes](#file-attributes)
         * [Curation Attributes](#curation-attributes)
         * [AdditionalInformation Attributes](#additionalinformation-attributes)
      * [Example of Local Metadata](#example-of-local-metadata)
      * [Example of Remote Metadata](#example-of-remote-metadata)
      * [References](#references)

<!--te-->

# Assets Metadata Ontology

`version 0.3`

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

### Local Metadata

Metadata is first created by the publisher of the asset. The publisher has knowledge of the file URLs, and they are stored in plaintext in the `files` object. This initial metadata is the "local metadata."

### Remote Metadata

A publisher runs Squid on their local machine. When they publish (register) an asset, they give their local metadata to their local Squid. Squid makes some changes and additions in the metadata, puts it into a DDO, and sends that DDO to a metadata store (Aquarius).

Aquarius may also make some changes and additions to the metadata, such as the `datePublished` or parts of the `curation` object. The metadata that finally gets stored by Aquarius is the "remote metadata."

Note: [OEP-11](11) describes the publishing flow in more detail.

## Metadata Attributes

A `metadata` object has the following attributes, all of which are objects.
Their contents are detailed below.

Attribute                 | Required |
--------------------------|----------|
**base**                  | Yes      |
**curation**              | (remote) |
**additionalInformation** | No       |

### Base Attributes

A `base` object has the following attributes.
Not all are required.
Some are required by only the metadata store *(remote)* and others are mandatory for *(local)* metadata only. If required or not by both, they are marked with *Yes/No* in the *Required* column.

Attribute       |   Type        |   Required    | Description
----------------|---------------|---------------|----------------------
**name**        | Text          | Yes           | Descriptive name or title of the Asset.
**dateCreated** | DateTime      | Yes   | The date on which the asset was created by the originator. ISO 8601 format, Coordinated Universal Time, (`2019-01-31T08:38:32Z`).
**datePublished** | DateTime      | (remote)   | The date on which the asset DDO is registered into the metadata store (Aquarius)
**author**      | Text          | Yes           | Name of the entity generating this data (e.g. Tfl, Disney Corp, etc.).
**license**     | Text          | Yes           | Short name referencing the license of the asset (e.g. Public Domain, CC-0, CC-BY, No License Specified, etc. ). If it's not specified, the following value will be added: "No License Specified".
**price**       | Number        | Yes           | Price of the asset. If not specified, then the default is 0.
**files**       | Array of file objects | Yes   | See the section about **File Attributes** below.
**encryptedFiles** | Text       | (remote)      | Encrypted **files** object. [OEP-11](11) specifies how it is computed.
**checksum**    | Text          | (remote)      | [OEP-7](7) specifies how the checksum is computed.
**categories**  | Array of Text | No            | Optional array of categories associated to the Asset.
**tags**        | Array of Text | No            | Keywords or tags used to describe this content. Multiple entries in a keyword list are typically delimited by commas. Empty by default.
**type**        | Text          | No            | Type of the Asset. Helps to filter by the type of asset. It could be for example ("dataset", "algorithm", "container", "workflow", "other"). It's up to the PROVIDER or MARKETPLACE to use a different list of types or not use it.
**description** | Text          | No            | Details of what the resource is. For a dataset, this attribute explains what the data represents and what it can be used for.
**copyrightHolder**| Text       | No            | The party holding the legal copyright. Empty by default.
**workExample** | Text          | No            | Example of the concept of this asset. This example is part of the metadata, not an external link.
**links**       | Array of Link | No            | Mapping of links for data samples, or links to find out more information. Links may be to either a URL or another Asset. We expect marketplaces to converge on agreements of typical formats for linked data: The Ocean Protocol itself does not mandate any specific formats as these requirements are likely to be domain-specific.
**inLanguage**  | Text          | No            | The language of the content. Please use one of the language codes from the [IETF BCP 47 standard](https://tools.ietf.org/html/bcp47).

#### File Attributes

A file object has the following attributes,
with the details necessary to consume and validate the data.

| Attribute         | Required | Description                                         |
| ----------------- | -------- | --------------------------------------------------- |
| **url**           | (local)  | Content URL. Omitted from the remote metadata. |
| **index**         | yes      | Index number starting from 0 of the file. |
| **contentType**   | yes      | File format. |
| **checksum**      | no       | Checksum of the file using your preferred format (i.e. MD5). Format specified in **checksumType**. If it's not provided can't be validated if the file was not modified after registering. |
| **checksumType**  | no       | Format of the provided checksum. Can vary according to server (i.e Amazon vs. Azure) |
| **contentLength** | no       | Size of the file in bytes.                        |
| **encoding**      | no       | File encoding (e.g. UTF-8). |
| **compression**   | no       | File compression (e.g. no, gzip, bzip2, etc). |
| **resourceId**    | no       | Remote identifier of the file in the external provider. It is typically the remote id in the cloud provider. |

### Curation Attributes

A `curation` object has the following attributes.

Attribute       |   Type           |   Required    | Description
----------------|------------------|---------------|----------------------
**rating**      | Number (decimal) | Yes           | Decimal value between 0 and 1. 0 is the default value.
**numVotes**    | Integer          | Yes           | Number of votes. 0 is the default value.
**schema**      | Text             | No            | Schema applied to calculate the rating.
**isListed**    | Boolean          | No            | Flag unsuitable content. False by default. If it's true, the content must not be returned.

### AdditionalInformation Attributes

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

## Example of Local Metadata

```json
{
  "base": {
    "name": "Madrid Weather forecast",
    "description": "Wheater forecast of Europe/Madrid in XML format",
    "dateCreated": "2019-05-16T12:36:14.535Z",
    "author": "Norwegian Meteorological Institute",
    "type": "dataset",
    "license": "Public Domain",
    "copyrightHolder": "Norwegian Meteorological Institute",
    "files": [
      {
        "url": "https://example-url.net/weather/forecast/madrid/350750305731.xml",
        "contentLength": "0",
        "contentType": "text/xml",
        "compression": "none",
        "index": 0
      }
    ],
    "categories": [
      "Other"
    ],
    "links": [],
    "tags": "",
    "price": 0
  },
  "additionalInformation": {
    "updateFrequency": null,
    "structuredMarkup": []
  }
}
```

## Example of Remote Metadata

Similarly, this is how the metadata file would look as a response to querying Aquarius (remote metadata). Note that `url` is removed from all objects in the `files` array, and `encryptedFiles` & `curation` are added.

```json
{
  "curation": {
    "rating": 0,
    "numVotes": 0,
    "isListed": true
  },
  "base": {
    "name": "Madrid Weather forecast",
    "description": "Wheater forecast of Europe/Madrid in XML format",
    "dateCreated": "2019-05-16T12:36:14.535Z",
    "author": "Norwegian Meteorological Institute",
    "type": "dataset",
    "license": "Public Domain",
    "copyrightHolder": "Norwegian Meteorological Institute",
    "files": [
      {
        "contentLength": "0",
        "contentType": "text/xml",
        "compression": "none",
        "index": 0
      }
    ],
    "categories": [
      "Other"
    ],
    "links": [],
    "tags": "",
    "price": 0,
    "encryptedFiles": "0x7a0d1c66ae861…df43aa9",
    "checksum": "d7296ccaaec630452be65a13ea1d2d750f071b6f50b779e99cc9adf05faebfca",
    "datePublished": "2019-05-16T12:41:01Z"
  },
  "additionalInformation": {
    "updateFrequency": null,
    "structuredMarkup": []
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
