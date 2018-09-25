```
shortname: 8/ASSET-DDO
name: Assets Metadata Ontology
type: Standard
status: Raw
editor: Aitor Argomaniz <aitor@oceanprotocol.com>
contributors:
```


Table of Contents
=================

   * [Table of Contents](#table-of-contents)
   * [Assets Metadata Ontology](#assets-metadata-ontology)
      * [Change Process](#change-process)
      * [Language](#language)
      * [Motivation](#motivation)
      * [Base attributes](#base-attributes)
      * [Curation attributes](#curation-attributes)
      * [Additional Information](#additional-information)
      * [Example](#example)
      * [References](#references)



# Assets Metadata Ontology

This OEP doesn't detail the exact method of registering ASSETS on-chain or publishing metadata in a metadata store.

Each Ocean Asset (dataset, algorithm, etc.) must have a base ontology associated with it.
This metadata, also called **DID descriptor objects (DDO)**, is stored in the OCEANDB and can be accessed using the **Decentralized ID (DID)**.
Assets without proper descriptive metadata can have poor visibility and discoverability.

The Asset ontology is based in the public schema.org [DataSet schema](https://schema.org/Dataset).


## Change Process

This document is governed by the [2/COSS](../2/README.md) (COSS).


## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.


## Motivation

The main motivations of this OEP are:

* Specify the common attributes that HAVE to be added in any ASSET DDO stored in the system
* Normalize the attributes to use in any curation process, allowing to have a common structure to sort and filter the DDO's
* Identify the recommended additional attributes that SHOULD be included in a DDO to facilitate the ASSETS search
* Provide an example of a possible structured ASSET DDO and additional links for reference


## Base attributes

Base attributes are always part of the DDO schema. Those attributes could be mandatory if they need to be completed by the publisher or can be stored empty. The following attributes are included as part of the Asset Metadata:


Attribute       |   Type        |   Required    | Description
----------------|---------------|---------------|----------------------
**name**        | Text          | Yes           | Descriptive name of the Asset
**type**        | Text          | Yes           | Type of the Asset. Helps to filter by kind of asset, initially ("dataset", "algorithm", "container", "workflow", "other")
**description** | Text          | No            | Details of what the resource is. For a data set explain what the data represents and what it can be used for
**dateCreated** | DateTime      | Yes           | The date on which  was created or was added
**size**        | Text          | Yes           | Size of the asset (e.g. 18mb). In the absence of a unit (mb, kb etc.), KB will be assumed
**author**      | Text          | Yes           | Name of the entity generating this data (e.g. Tfl, Disney Corp, etc.)
**license**     | Text          | Yes           | Short name referencing to the license of the asset (e.g. Public Domain, CC-0, CC-BY, No License Specified, etc. ). If it's not specified, the following value will be added: "No License Specifiedified"
**copyrightHolder**| Text       | No            | The party holding the legal copyright. Empty by default
**encoding**    | Text          | No            | File encoding (e.g. UTF-8)
**compression** | Text          | No            | File compression (e.g. no, gzip, bzip2, etc)
**contentType** | Text          | Yes           | File format if applicable
**workExample** | Text          | No            | Example of the concept of this asset. This example is part of the metadata, not an external link.
**contentUrls** | Text          | Yes           | List of content urls resolving the ASSET files
**links**       | Text       | No               | Mapping of links for data samples, or links to find out more information. The key represents the topic of the link, the value is the proper link
**inLanguage**  | Text          | No            | The language of the content or performance or used in an action. Please use one of the language codes from the [IETF BCP 47 standard](https://tools.ietf.org/html/bcp47)
**tags**        | Text          | No            | Keywords or tags used to describe this content. Multiple entries in a keywords list are typically delimited by commas. Empty by default
**price**       | Number        | Yes           | Price of the asset. If not specified would be 0.


## Curation attributes

In order to normalize the different possible rating attributes after a process of curation, this is the normalize list of attributes to define:

Attribute       |   Type        |   Required    | Description
----------------|---------------|---------------|----------------------
**rating**     | Number (decimal)       | Yes              | Decimal values between 0 and 1. 0 is the default value
**numVotes**    | Integer       | Yes              | Number of votes. 0 is the default value
**schema**      | Text          | No              | Schema applied to calculate the rating



## Additional Information

These are examples of attributes that can enhance the discoverability of a resource:

| Attribute         | Description                                                                                                                  |
| -                 | -                                                                                                                            |
| checksum          | Checksum of attributes to be able to compare if there are changes in the asset that you are purchasing.                      |
| sla               | Service Level Agreement                                                                                                      |
| industry          |                                                                                                                              |
| category          | can be assigned to a category in addition to having tags                                                                     |
| updateFrequency   | how often are updates expected (seldome, annual, quarterly, etc.), or is the resource static (never expected to get updated) |
| termsOfService    |                                                                                                                              |
| privacy           |                                                                                                                              |
| keyword           | A list of keywords/tags describing a dataset                                                                                 |
| structured-markup | A link to machine readable structured markup (such as ttl/json-ld/rdf) describing the dataset                                |
|                   |                                                                                                                              |

Additional attributes are totally free to add and can be defined by the publisher of the DDO, in addition to the base attributes


## Example

Here a representation of an example Asset using the schema described:

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
            {"sample1": "http://data.ceda.ac.uk/badc/ukcp09/data/gridded-land-obs/gridded-land-obs-daily/"},
            {"sample2": "http://data.ceda.ac.uk/badc/ukcp09/data/gridded-land-obs/gridded-land-obs-averages-25km/"},
            {"fieldsDescription": "http://data.ceda.ac.uk/badc/ukcp09/"}
         ],
        "inLanguage": "en",
        "tags": "weather, uk, 2011, temperature, humidity",
        "price": 10

    },
    "curation": {
        "rating": 0.93
        "numVotes": 123,
        "schema": "Binary Votting"
    },
    "additionalInformation" : {
        "updateFrecuency": "yearly",
        "structuredMarkup" : [ { "uri" : "http://skos.um.es/unescothes/C01194/jsonld", "mediaType" : "application/ld+json"},
                               { "uri" : "http://skos.um.es/unescothes/C01194/turtle", "mediaType" : "text/turtle"}]
    }
}
```




## References

[Schema.org](https://schema.org/) is a collaborative, community activity with a mission to create, maintain, and promote schemas for structured data on the Internet,
Data types use the [Schema.org primitive data types](https://schema.org/DataType).

Schemas:

* DataSet - https://schema.org/Dataset
* FileSize - https://schema.org/fileSize
* Common license types for datasets - https://help.data.world/hc/en-us/articles/115006114287-Common-license-types-for-datasets






