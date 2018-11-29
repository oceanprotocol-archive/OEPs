```
shortname: 8/ASSET-DDO
name: Assets Metadata Ontology
type: Standard
status: Raw
editor: Aitor Argomaniz <aitor@oceanprotocol.com>
contributors: Kiran Karkera <kiran.karkera@dex.sg>, Enrique Ruiz <enrique@oceanprotocol.com>, Mike Anderson <mike.anderson@dex.sg>, Matthias Kretschmann <matthias@oceanprotocol.com>
```

**Table of Contents**

<!--ts-->

   * [Assets Metadata Ontology](#assets-metadata-ontology)
      * [Change Process](#change-process)
      * [Language](#language)
      * [Motivation](#motivation)
      * [Base Attributes](#base-attributes)
      * [Curation Attributes](#curation-attributes)
      * [Additional Information](#additional-information)
      * [Example](#example)
      * [References](#references)

<!--te-->

# Assets Metadata Ontology

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

## Base Attributes

All base attributes MUST be included in the Asset Metadata. The stored _values_ can be empty. The following are the base attributes:

Attribute       |   Type        |   Required    | Description
----------------|---------------|---------------|----------------------
**name**        | Text          | Yes           | Descriptive name of the Asset.
**type**        | Text          | Yes           | Type of the Asset. Helps to filter by the type of asset, initially ("dataset", "algorithm", "container", "workflow", "other").
**description** | Text          | No            | Details of what the resource is. For a data set, this attribute explains what the data represents and what it can be used for.
**dateCreated** | DateTime      | Yes           | The date on which the asset was created or was added.
**size**        | Text          | Yes           | Size of the asset (e.g. 18MB). In the absence of a unit (MB, kB etc.), kB will be assumed.
**author**      | Text          | Yes           | Name of the entity generating this data (e.g. Tfl, Disney Corp, etc.).
**license**     | Text          | Yes           | Short name referencing the license of the asset (e.g. Public Domain, CC-0, CC-BY, No License Specified, etc. ). If it's not specified, the following value will be added: "No License Specified".
**copyrightHolder**| Text       | No            | The party holding the legal copyright. Empty by default.
**encoding**    | Text          | No            | File encoding (e.g. UTF-8).
**compression** | Text          | No            | File compression (e.g. no, gzip, bzip2, etc).
**contentType** | Text          | Yes           | File format, if applicable.
**workExample** | Text          | No            | Example of the concept of this asset. This example is part of the metadata, not an external link.
**contentUrls** | Text          | Yes           | List of content URLs resolving the Asset files.
**links**       | Array of Link | No            | Mapping of links for data samples, or links to find out more information. Links may be to either a URL or another Asset. We expect marketplaces to converge on agreements of typical formats for linked data: The Ocean Protocol itself does not mandate any specific formats as these requirements are likely to be domain-specific.
**inLanguage**  | Text          | No            | The language of the content. Please use one of the language codes from the [IETF BCP 47 standard](https://tools.ietf.org/html/bcp47).
**tags**        | Text          | No            | Keywords or tags used to describe this content. Multiple entries in a keyword list are typically delimited by commas. Empty by default.
**price**       | Number        | Yes           | Price of the asset. If not specified, then the default is 0.

## Curation Attributes

To normalize the different possible rating attributes after a curation process, this is the normalized list of curation attributes:

Attribute       |   Type           |   Required    | Description
----------------|------------------|---------------|----------------------
**rating**      | Number (decimal) | Yes           | Decimal value between 0 and 1. 0 is the default value.
**numVotes**    | Integer          | Yes           | Number of votes. 0 is the default value.
**schema**      | Text             | No            | Schema applied to calculate the rating.

## Additional Information

These are examples of attributes that can enhance the discoverability of a resource:

| Attribute             | Description                                                                                                                  |
| -                     | -                                                                                                                            |
| **checksum**          | Checksum of attributes to be able to compare if there are changes in the asset that you are purchasing.                      |
| **sla**               | Service Level Agreement.                                                                                                      |
| **industry**          |                                                                                                                              |
| **category**          | Can be assigned to a category in addition to having tags.                                                                     |
| **updateFrequency**   | An indication of update latency - i.e. How often are updates expected (seldom, annually, quarterly, etc.), or is the resource static that is never expected to get updated. |
| **termsOfService**    |                                                                                                                              |
| **privacy**           |                                                                                                                              |
| **keyword**           | A list of keywords/tags describing a dataset.                                                                                 |
| **structured-markup** | A link to machine-readable structured markup (such as ttl/json-ld/rdf) describing the dataset.                                |

The publisher of a DDO MAY add additional attributes (i.e. in addition to those listed above).

## Example

Here is an example of an Asset Metadata object following the above-described schema:

```json
{
    "base": {
        "name": "UK Weather information 2011",
        "type": "dataset",
        "description": "Weather information of UK including temperature and humidity",
        "size": "3.1gb",
        "dateCreated": "2012-02-01T10:55:11+00:00",
        "author": "Met Office",
        "license": "CC-BY",
        "copyrightHolder": "Met Office",
        "encoding": "UTF-8",
        "compression": "zip",
        "contentType": "text/csv",
        "workExample": "stationId,latitude,longitude,datetime,temperature,humidity\n
                        423432fsd,51.509865,-0.118092,2011-01-01T10:55:11+00:00,7.2,68",
        "contentUrls": ["https://testocnfiles.blob.core.windows.net/testfiles/testzkp.zip"],
        "links": [
            {
                "name": "Sample of Asset Data",
                "type": "sample",
                "url": "https://foo.com/sample.csv"
            },
            {
                "name": "Data Format Definition",
                "type": "format",
                "AssetID": "4d517500da0acb0d65a716f61330969334630363ce4a6a9d39691026ac7908ea"
            }
        ],
        "inLanguage": "en",
        "tags": "weather, uk, 2011, temperature, humidity",
        "price": 10
    },
    "curation": {
        "rating": 0.93,
        "numVotes": 123,
        "schema": "Binary Voting"
    },
    "additionalInformation" : {
        "updateFrecuency": "yearly",
        "structuredMarkup": [
            {
                "uri": "http://skos.um.es/unescothes/C01194/jsonld",
                "mediaType": "application/ld+json"
            },
            {
                "uri": "http://skos.um.es/unescothes/C01194/turtle",
                "mediaType": "text/turtle"
            }
        ]
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
