```
shortname: 8/ASSET-DDO
name: Assets Metadata Ontology
type: Standard
status: Raw
editor: Aitor Argomaniz <aitor@oceanprotocol.com>
contributors: Kiran Karkera <kiran.karkera@dex.sg>, Enrique Ruiz <enrique@oceanprotocol.com>, Mike Anderson <mike.anderson@dex.sg>, Matthias Kretschmann <matthias@oceanprotocol.com>
```


Table of Contents
=================

   * [Table of Contents](#table-of-contents)
   * [Asset Metadata](#asset-metadata)
      * [Change Process](#change-process)
      * [Language](#language)
      * [Motivation](#motivation)
      * [Base attributes](#base-attributes)
        * [Additional Information](#additional-information)
        * [Links](#links)
      * [Data asset attributes](#data-asset-attributes)
      * [Invokable service attributes](#invokable-service-attributes)
      * [Bundle attributes](#bundle-attributes)
      * [Example](#example)
      * [References](#references)



# Asset Metadata 

Ocean has a core mission of making data assets visible and discoverable, with a decentralised protocol for data exchange.

As such, each Ocean Asset (dataset, algorithm, etc.) has Asset Metadata associated with it. Asset Metadata is created by the asset publisher, and hashed to ensure integrity and ensure that the asset can be subsequently referenced as part of the provenance of other assets.

Assets without proper descriptive metadata can have poor visibility and discoverability, so it is generally in the publisher's interest to ensure good metadata is made available.

This OEP doesn't detail the exact method of registering or publishing metadata in a metadata store.


## Change Process

This document is governed by the [2/COSS](../2/README.md) (COSS).


## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.


## Motivation

The main motivations of this OEP are:

* Establish a standard Asset Metadata format to allow interoperability between participants in the Ocean ecosystem
* Specify the common attributes that HAVE to be added in Asset Metadata
* Identify the recommended additional attributes that SHOULD be included in Asset Metadata to facilitate the ASSETS search
* Provide an example of a possible structured Asset Metadata and additional links for reference


## Base attributes

Base attributes are always part of the Asset Metadata schema. Those attributes could be mandatory if they need to be completed by the publisher or can be stored empty. The following attributes are included as part of the Asset Metadata:


Attribute       |   Type        |   Required    | Description
----------------|---------------|---------------|----------------------
**name**        | Text          | Yes           | Descriptive name of the Asset
**type**        | Text          | Yes           | Type of the Asset. Helps to filter by kind of asset, initially ("data", "invoke", "bundle", "algorithm", "container", "workflow", "other")
**description** | Text          | No            | Details of what the resource is. For a data set explain what the data represents and what it can be used for
**dateCreated** | DateTime      | Yes           | The date on which  was created or was added
**author**      | Text          | Yes           | Name of the entity generating this data (e.g. Tfl, Disney Corp, etc.)
**license**     | Text          | Yes           | Short name referencing to the license of the asset (e.g. Public Domain, CC-0, CC-BY, No License Specified, etc. ). If it's not specified, the following value will be added: "No License Specified"
**copyrightHolder**| Text       | No            | The party holding the legal copyright. Empty by default
**workExample** | Text          | No            | Example of the concept of this asset. This example is part of the metadata, not an external link.
**links**       | Array of Link | No            | Mapping of links for data samples, or links to find out more information. 
**inLanguage**  | Text          | No            | The language of the content or performance or used in an action. Please use one of the language codes from the [IETF BCP 47 standard](https://tools.ietf.org/html/bcp47)
**tags**        | Array of Text | No            | Keywords or tags used to describe this content. Multiple entries in a keywords list are typically delimited by commas. Empty by default
**additionalInformation** | Map | No            | Additional JSON content at discretion of publisher

### Additional Information

These are examples of attributes that can enhance the discoverability of a resource:

| Attribute         | Description                                                                                                                  |
| -                 | -                                                                                                                            |
| sla               | Service Level Agreement                                                                                                      |
| industry          |                                                                                                                              |
| category          | can be assigned to a category in addition to having tags                                                                     |
| updateFrequency   | how often are updates expected (seldome, annual, quarterly, etc.), or is the resource static (never expected to get updated) |
| termsOfService    |                                                                                                                              |
| privacy           |                                                                                                                              |
| keyword           | A list of keywords/tags describing a dataset                                                                                 |
| structured-markup | A link to machine readable structured markup (such as ttl/json-ld/rdf) describing the dataset                                |                                                                                                                  |

Additional attributes are totally free to add and can be defined by the publisher, in addition to the base attributes


### Links

An array of Links can be provided to give supplementary information about an Asset, e.g.

```json
[
	{
		"name" : "Sample of Asset Data",
		"type" : "sample",
		"url": "https://foo.com/sample.csv"
	}
	{
		"name" : "Data Format Definition",
		"type" : "format",
		"assetID: "4d517500da0acb0d65a716f61330969334630363ce4a6a9d39691026ac7908ea"
	}	
]
```

Links may be to either a URL or another Asset. We expect tribes and/or marketplaces to converge
on agreements of typical formats for linked data: The Ocean protocol itself does not mandate any
specific formats as requirements are likely to be domain-specific.

## Data asset attributes

In addition to the base attributes, the following Attributes are defined for data assets only (with type: "dataset")

Attribute       |   Type        |   Required    | Description
----------------|---------------|---------------|----------------------
**size**        | Text          | Yes           | Size of the asset (e.g. 18mb). In the absence of a unit (mb, kb etc.), KB will be assumed
**encoding**    | Text          | No            | File encoding (e.g. UTF-8)
**compression** | Text          | No            | File compression (e.g. no, gzip, bzip2, etc)
**contentType** | Text          | Yes           | File format if applicable
**contentUrls** | Text          | No           | List of content urls resolving the ASSET files
**contentHash** | Text          | No           | keccak256 hash of asset data. Required if publisher wishes to offer integrity checks

## Invokable service attributes

In addition to the base attributes, the following Attributes are defined for invokable services only (with type: "invoke")

Invokable services are defined in greater detail in OEP6.

Attribute       |   Type        |   Required    | Description
----------------|---------------|---------------|----------------------
**params**        | Text          | Yes           | A list of parameters accepted by the invokable service

## Bundle attributes

In addition to the base attributes, the following Attributes are defined for bundles of assets only (with type: "bundle")

Attribute       |   Type        |   Required    | Description
----------------|---------------|---------------|----------------------
**contents**    | Map of String -> Content          | Yes           | A list of contents of this bundle

Content records are specified as a JSON map with the following attributes

Attribute      |   Type        |   Required    | Description
---------------|---------------|---------------|----------------------
**assetID**    | String (64)   | Yes           | A hex string containing the Asset ID of the content


Example:

```json
{ 
  "name": "Pyrotech firework safety data",
  "type": "bundle",
  "contents": {
  	"test":   {"assetID": "26cb1a92e8a6b52e47e6e13d04221e9b005f70019e21c4586dad3810d46220135"}
  	"train":  {"assetID": "503a7b959f91ac691a0881ee724635427ea5f3862aa105040e30a0fee50cc1a00"}
  	"verify": {"assetID": "e2f910c3f44126323ace27daf9a6e18ab0cbcc3ab9fa74a7c9462d7e8247f8811"}
  }
}
```  



## Example

Here is a representation of an example Asset using the schema described:

```json
{
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
		    "name" : "Sample of Asset Data",
		    "type" : "sample",
		    "url": "https://foo.com/sample.csv"
	    }
	    {
		    "name" : "Data Format Definition",
		    "type" : "format",
		    "assetID: "4d517500da0acb0d65a716f61330969334630363ce4a6a9d39691026ac7908ea"
	    }	
    ],
    "inLanguage": "en",
    "tags": ["weather", "uk", "2011", "temperature", "humidity"],

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

[Schema.org](https://schema.org/) is a collaborative, community activity with a mission to create, maintain, and promote schemas for structured data on the Internet,
Data types use the [Schema.org primitive data types](https://schema.org/DataType).

The Asset ontology for datasets is based in the public schema.org [DataSet schema](https://schema.org/Dataset).

Schemas:

* DataSet - https://schema.org/Dataset
* FileSize - https://schema.org/fileSize
* Common license types for datasets - https://help.data.world/hc/en-us/articles/115006114287-Common-license-types-for-datasets

