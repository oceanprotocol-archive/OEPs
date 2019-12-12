# OEP-8: Assets Metadata Ontology

```
shortname:    8/ASSET-DDO
name:         Assets Metadata Ontology
type:         Standard
status:       Raw
version:      0.4
editor:       Aitor Argomaniz <aitor@oceanprotocol.com>
contributors: Enrique Ruiz <enrique@oceanprotocol.com>,
              Matthias Kretschmann <matthias@oceanprotocol.com>,
              Jose Pablo Fernandez <jose@oceanprotocol.com>,
              Marcus Jones <marcus@oceanprotocol.com>,
              Troy McConaghy <troy@oceanprotocol.com>,
              Alex Coseru <alex@oceanprotocol.com>
```

**Table of Contents**

- [Motivation](#motivation)
- [Life Cycle of Metadata](#life-cycle-of-metadata)
  - [Local Metadata](#local-metadata)
  - [Remote Metadata](#remote-metadata)
- [Metadata Attributes](#metadata-attributes)
  - [Main Attributes](#main-attributes)
    - [File Attributes](#file-attributes)
  - [Additional Attributes](#additional-attributes)
    - [Other Suggested Additional Attributes](#other-suggested-additional-attributes)
  - [Curation Attributes](#curation-attributes)
- [Example of Local Metadata](#example-of-local-metadata)
- [Example of Remote Metadata](#example-of-remote-metadata)
  - [Specific attributes per asset type](#specific-attributes-per-asset-type)
    - [Algorithm attributes](#algorithm-attributes)
    - [Workflow attributes](#workflow-attributes)
    - [Service attributes](#service-attributes)
- [References](#references)
- [Change Process](#change-process)
- [Language](#language)

---

## Motivation

Every asset (dataset, algorithm, etc.) in the Ocean Network has an associated Decentralized Identifier (DID) and DID document / DID Descriptor Object (DDO). Because assets without proper descriptive metadata have poor visibility and discoverability.

See [OEP 7/DID](../7/README.md) for information about the overall structure of Ocean DDOs and DIDs.

This OEP is about one particular part of Ocean DDOs: the asset metadata, a JSON object with information about the asset.

This OEP defines the assets metadata ontology, i.e. the schema for the asset metadata. It's based on the public schema.org [DataSet schema](https://schema.org/Dataset).

This OEP doesn't detail the exact method of registering assets on-chain or storing DDOs.

The main motivations of this OEP are to:

* Specify the common attributes that MUST be included in any asset metadata stored in the Ocean Network
* Normalize the attributes to use in any curation process, to provide a common structure to sort and filter the DDOs
* Identify the recommended additional attributes that SHOULD be included in a DDO to facilitate asset search
* Provide an example of an asset metadata object and additional links for reference

## Life Cycle of Metadata

### Local Metadata

Metadata is first created by the publisher of the asset. The publisher has knowledge of the file URLs, and they are stored in plaintext in the `files` object. This initial metadata is the _local metadata_.

### Remote Metadata

A publisher publishes (registers) an asset using [Squid](https://docs.oceanprotocol.com/concepts/components/#squid-libraries), which might be running on their local machine or remotely. When they do, the local metadata is passed to Squid, which makes some changes and additions in the metadata, puts it into a DDO, and sends that DDO to a metadata store (Aquarius).

Aquarius may also make some changes and additions to the metadata, such as the `datePublished` or parts of the `curation` object. The metadata that finally gets stored by Aquarius is the _remote metadata_.

> A marketplace can and might also act as a publisher. [OEP-11](11) describes the publishing flow in more detail.

## Metadata Attributes

An asset is the representation of different type of resources in Ocean Protocol. Typically can asset could be one of the following asset types:

* _Dataset_. An asset representing a dataset or data resource. It could be for example a CSV file or a multiple JPG files.
* _Algorithm_. An asset representing a piece of software. It could be a python script using tensorflow, a spark job, etc.
* _Workflow_. An asset representing in the metadata a list of tasks to accomplish with the intention of process data, typically relating datasets as input and algorithms. 
* _Service_. An asset representing a service exposed by a provider. Typically a REST web service that a provider want to control the access using SEA.

Each kind of asset require a different subset of metadata attributes. The distintion between the type of asset (dataset, algorithm, etc.) is given by the attribute `DDO.services["metadata"].main.type`

A `metadata` object has the following attributes, all of which are objects.

Attribute                   | Required | Description
----------------------------|----------|----------|
**`main`**                  | Yes      | Main attributes used to calculate the service checksum |
**`curation`**              | (remote) | Curation attributes
**`additionalInformation`** | No       | Optional attributes
**`encryptedFiles`**        | (remote) | Encrypted string of the `attributes.main.files` object.
**`encryptedServices`**      | (remote) | Encrypted string of the `attributes.main.services` object.

The `main`, `curation` and `additionalInformation` attributes are independent of the asset type, all assets have those metadata sections.

### Main Attributes

**This list of attributes can't be modified after creation**, because these are considered as the metadata essence of the asset created. This information is used to calculate the unique checksum of the asset. If any change would be necessary in the following attributes, it would be necessary to create a new asset derived from the existing one.

The `main` object has the following attributes, not all are required. Some are required by only the metadata store (_remote_) and others are mandatory for _local_ metadata only. If required or not by both, they are marked with _Yes/No_ in the _Required_ column.

Attribute       |   Type        |   Required    | Description
----------------|---------------|---------------|----------------------
**`name`**        | Text          | Yes           | Descriptive name or title of the asset.
**`type`**        | Text          | Yes            | Type of the asset. Helps to filter by the type of asset. It could be for example ("dataset", "algorithm", "container", "workflow", "service", "other").
**`dateCreated`** | DateTime      | Yes   | The date on which the asset was created by the originator. ISO 8601 format, Coordinated Universal Time, e.g. `2019-01-31T08:38:32Z`.
**`datePublished`** | DateTime      | (remote)   | The date on which the asset DDO is registered into the metadata store (Aquarius)
**`author`**      | Text          | Yes           | Name of the entity generating this data (e.g. Tfl, Disney Corp, etc.).
**`license`**     | Text          | Yes           | Short name referencing the license of the asset (e.g. Public Domain, CC-0, CC-BY, No License Specified, etc. ). If it's not specified, the following value will be added: "No License Specified".
**`price`**       | String        | Yes           | Price of the asset in _vodka_ (_attoOCEAN_). It must be an integer encoded as a string, e.g. `"123000000000000000000"`.
**`files`**       | Array of files object | Yes     | Array of `File` objects including the encrypted file urls. Further metadata about each file is stored, see [File Attributes](#file-attributes)


#### File Attributes

File attributes are a subset of the `main` section.

A file object has the following attributes, with the details necessary to consume and validate the data.

| Attribute         | Required | Description                                         |
| ----------------- | -------- | --------------------------------------------------- |
| **`url`**           | (local)  | Content URL. Omitted from the remote metadata. Supports `http(s)://` and `ipfs://` URLs. |
| **`name`**          | no       | File name. |
| **`index`**         | yes      | Index number starting from 0 of the file. |
| **`contentType`**   | yes      | File format. |
| **`checksum`**      | no       | Checksum of the file using your preferred format (i.e. MD5). Format specified in `checksumType`. If it's not provided can't be validated if the file was not modified after registering. |
| **`checksumType`**  | no       | Format of the provided checksum. Can vary according to server (i.e Amazon vs. Azure) |
| **`contentLength`** | no       | Size of the file in bytes.                        |
| **`encoding`**      | no       | File encoding (e.g. UTF-8). |
| **`compression`**   | no       | File compression (e.g. no, gzip, bzip2, etc). |
| **`encrypted`**     | no       | Boolean. Is the file encrypted? If is not set is assumed the file is not encrypted |
| **`encryptionMode`**| no       | Encryption mode used. Just valid if `encrypted=true` |
| **`resourceId`**    | no       | Remote identifier of the file in the external provider. It is typically the remote id in the cloud provider. |
| **`attributes`**    | no       | Key-Value hash map with additional attributes describing the asset file. It could include details like the Amazon S3 bucket, region, etc. |

### Additional Attributes

All the additional information will be stored as part of the `additionalInformation` section.

| Attribute         | Type | Required                                         |
| ----------------- | -------- | --------------------------------------------------- |
| **`categories`**  | Array of Text | No            | Optional array of categories associated to the asset. |
| **`tags`**        | Array of Text | No            | Array of keywords or tags used to describe this content. Empty by default. |
| **`description`** | Text          | No            | Details of what the resource is. For a dataset, this attribute explains what the data represents and what it can be used for. |
| **`copyrightHolder`**| Text       | No            | The party holding the legal copyright. Empty by default. |
| **`workExample`** | Text          | No            | Example of the concept of this asset. This example is part of the metadata, not an external link. |
| **`links`**       | Array of Link | No            | Mapping of links for data samples, or links to find out more information. Links may be to either a URL or another Asset. We expect marketplaces to converge on agreements of typical formats for linked data: The Ocean Protocol itself does not mandate any specific formats as these requirements are likely to be domain-specific. The links array can be an empty array, but if there is a link object in it, then an "url" is required in that link object. |
| **`inLanguage`**  | Text          | No            | The language of the content. Please use one of the language codes from the [IETF BCP 47 standard](https://tools.ietf.org/html/bcp47). |

#### Other Suggested Additional Attributes

These are examples of attributes that can enhance the discoverability of a resource:

| Attribute             | Description                                                                                                                  |
| -                     | -                                                                                                                            |
| **`sla`**               | Service Level Agreement.                                                                                                      |
| **`industry`**          |                                                                                                                              |
| **`updateFrequency`**   | An indication of update latency - i.e. How often are updates expected (seldom, annually, quarterly, etc.), or is the resource static that is never expected to get updated. |
| **`termsOfService`**    |                                                                                                                              |
| **`privacy`**           |                                                                                                                              |
| **`keyword`**           | A list of keywords/tags describing a dataset.                                                                                 |
| **`structuredMarkup`** | A link to machine-readable structured markup (such as ttl/json-ld/rdf) describing the dataset.                                |

The publisher of a DDO MAY add additional attributes or change the above object definition.

### Curation Attributes

A `curation` object has the following attributes.

Attribute       |   Type           |   Required    | Description
----------------|------------------|---------------|----------------------
**`rating`**      | Number (decimal) | Yes           | Decimal value between 0 and 1. 0 is the default value.
**`numVotes`**    | Integer          | Yes           | Number of votes. 0 is the default value.
**`schema`**      | Text             | No            | Schema applied to calculate the rating.
**`isListed`**    | Boolean          | No            | Flag unsuitable content. False by default. If it's true, the content must not be returned.

## Example of Local Metadata

```json
{  
  "main": {
    "name": "Madrid Weather forecast",
    "dateCreated": "2019-05-16T12:36:14.535Z",
    "author": "Norwegian Meteorological Institute",
    "type": "dataset",
    "license": "Public Domain",
    "price": "123000000000000000000",
    "files": [  
      {
        "index": 0,
        "url": "https://example-url.net/weather/forecast/madrid/350750305731.xml",
        "contentLength": "0",
        "contentType": "text/xml",
        "compression": "none"
      }
    ],
    "services": [  
      {  
        "index": 0,
        "url": "https://my.service.inet:8080/api/v1/weather",
        "method": "POST",
        "auth": {  
          "user": "aitor",
          "password": "1234",
          "token": "89c06eb5a88f4bbbf4ac966d737593b36e61e885"
        },
        "description": "https://my.service.inet:8080/spec",
        "checksum": "859486596784567856758aaaa"
      }
    ]
  },
  "additionalInformation":{  
    "description": "Weather forecast of Europe/Madrid in XML format",
    "copyrightHolder": "Norwegian Meteorological Institute",
    "categories": ["Other"],
    "links": [],
    "tags": [],
    "updateFrequency": null,
    "structuredMarkup": []
  }
}
```

## Example of Remote Metadata

Similarly, this is how the metadata file would look as a response to querying Aquarius (remote metadata). Note that `url` is removed from all objects in the `files` array, and `encryptedFiles` & `curation` are added.

```json
{  
  "service": [
    {  
      "index": 0,
      "serviceEndpoint": "http://aquarius:5000/api/v1/aquarius/assets/ddo/{did}",
      "type": "metadata",
      "attributes": {  
        "main": {  
          "type": "dataset",
          "name": "Madrid Weather forecast",
          "dateCreated": "2019-05-16T12:36:14.535Z",
          "author": "Norwegian Meteorological Institute",
          "license": "Public Domain",
          "price": "123000000000000000000",
          "files":[  
            {  
              "contentLength": "0",
              "contentType": "text/xml",
              "compression": "none",
              "index": 0
            }
          ],          
          "datePublished": "2019-05-16T12:41:01Z"
        },
        "encryptedFiles": "0x7a0d1c66ae861…df43aa9",
        "curation":{  
          "rating": 1,
          "numVotes": 7,
          "schema": "BINARY",
          "isListed": true
        },
        "additionalInformation": {  
          "description": "Weather forecast of Europe/Madrid in XML format",
          "copyrightHolder": "Norwegian Meteorological Institute",
          "categories": ["Other"],
          "links": [],
          "tags": [],
          "updateFrequency": null,
          "structuredMarkup": []
        }
      }
    }
  ]
}
```

### Specific attributes per asset type

Depending on the asset type (dataset, algorithm, workflow, service), there are different metadata attributes supported:

#### Algorithm attributes

An asset of type `algorithm` has the following attributes:

| Attribute         | Required | Description                                         |
| ----------------- | -------- | --------------------------------------------------- |
| **`language`**      | yes      | Language used to implement the software |
| **`format`**        | no      | Packaging format of the software. |
| **`version`**       | no      | Version of the software. |
| **`entrypoint`**      | yes   | Path of the script inside of the software package used to install the dependencies and run it. |
| **`container`**      | yes   | Container image and tag to use |
| **`requirements`**  | no   | Array of software requirements objects. |

```json
{  
  "service":[  
    {  
      "index": 0,
      "serviceEndpoint": "http://aquarius:5000/api/v1/aquarius/assets/ddo/{did}",
      "type": "metadata",
      "attributes": {  
        "main": {  
          "type": "algorithm",
          "algorithm": {  
            "language": "scala",
            "format": "jar",
            "version": "0.1",
            "entrypoint": "ocean-entrypoint.sh",
            "container": "oceanprotocol/imageName:v1.0.0",
            "requirements": [  
              {  
                "requirement": "scala",
                "version": "2.12.8"
              },
              {  
                "requirement": "java",
                "version": "1.8"
              }
            ]
          }
        }
      }
    }
  ]
}
```

#### Workflow attributes

An asset of type `workflow` has the following attributes:

| Attribute         | Required | Description                                         |
| ----------------- | -------- | --------------------------------------------------- |
| **`stages`**        | yes      | Array of stages/steps of a workflow. It is required to have at least one stage. |
| **`stages.index`**  | yes      | Index number starting from 0 of the stage. |
| **`stages.stageType`**  | no      | Optional text with information about the type of stage (cleansing, filtering, etc.)  |
| **`stages.requirements`**| yes      | Object defining the system requirements of the stage to be executed |
| **`stages.requirements.container`**| yes      | Object defining the details of the container necessary to execute the stage |
| **`stages.requirements.container.image`**| yes      | Docker image to use |
| **`stages.requirements.container.tag`**| yes      | Docker image tag to use |
| **`stages.requirements.container.checksum`**| yes      | Checksum of the image and tag |
| **`stages.input`**  | yes      | Array of inputs of a stage |
| **`stages.input.index`**  | yes      | Index number starting from 0 of the input. The inputs will be given to the algorithm in the order defined in the index |
| **`stages.input.id`**  | yes      | DID of the Asset (dataset) |
| **`stages.transformation`**  | yes      | Object describing the transformation or computation phase made by an algorithm |
| **`stages.transformation.id`**  | yes      | DID of the Asset (algorithm) in charge of process the input data |
| **`stages.output`**  | yes      | Object including information to tag the output generated |
| **`stages.output.metadataUrl`**  | yes      | Url of the Metadata service (Aquarius) that will be used for publishing the metadata of the new asset |
| **`stages.output.secretStoreUrl`**  | yes      | Url of the Secret Store service used to encrypt the access urls |
| **`stages.output.accessProxyUrl`**  | yes      | Url of the server used to consume the asset (Brizo) |
| **`stages.output.metadata`**  | yes      | Object including all the metadata information that will be used to tag the new asset generated |

Example of workflow:

```json
{
  "service": [  
    {  
      "index": 0,
      "serviceEndpoint": "http://aquarius:5000/api/v1/aquarius/assets/ddo/{did}",
      "type": "metadata",
      "attributes": {  
        "main": {  
          "type":"workflow",
          "workflow": {  
            "stages": [  
              {  
                "index": 0,
                "stageType": "Filtering",
                "requirements": {  
                  "container": {  
                    "image": "tensorflow/tensorflow",
                    "tag": "latest",
                    "checksum": "sha256:cb57ecfa6ebbefd8ffc7f75c0f00e57a7fa739578a429b6f72a0df19315deadc"
                  }
                },
                "input": [  
                  {  
                    "index": 0,
                    "id": "did:op:12345"
                  },
                  {  
                    "index": 1,
                    "id": "did:op:67890"
                  }
                ],
                "transformation": {  
                  "id": "did:op:abcde"
                },
                "output": {  
                  "metadataUrl": "https://aquarius.net:5000/api/v1/aquarius/assets/ddo/",
                  "secretStoreUrl": "http://secretstore.org:12001",
                  "accessProxyUrl": "https://brizo.net:8030/api/v1/brizo/",
                  "metadata": {  
                    "title": "my filtered asset"
                  }
                }
              },
              {  
                "index": 1,
                "stageType": "Transformation",
                "requirements": {  
                  "container": {  
                    "image": "tensorflow/tensorflow",
                    "tag": "latest",
                    "checksum": "sha256:cb57ecfa6ebbefd8ffc7f75c0f00e57a7fa739578a429b6f72a0df19315deadc"
                  }
                },
                "input": [  
                  {  
                    "index": 0,
                    "previousStage": 0
                  }
                ],
                "transformation": {  
                  "id": "did:op:999999"
                },
                "output": {  
                  "metadataUrl": "https://aquarius.net:5000/api/v1/aquarius/assets/ddo/",
                  "secretStoreUrl": "http://secretstore.org:12001",
                  "accessProxyUrl": "https://brizo.net:8030/api/v1/brizo/",
                  "metadata": {}
                }
              }
            ]
          }
        }
      }
    }
  ]
}
```

#### Service attributes

An asset of type `service` has an entry named `services`, which is an array of the individual services or endpoints exposed as part of the service.

The attributes included in each item are the following:

| Attribute         | Required | Description                                         |
| ----------------- | -------- | --------------------------------------------------- |
| **`spec`**          | no       | Url to the web service specifications (Swagger, Documentation, etc.) |
| **`specChecksum`**  | no       | Checksum of the service specifications in SHA3 |
| **`definition`**    | yes      | Object containing the web service definition. This entry will be encrypted and the hash placed in the `base.encryptedServices` attribute. |
| **`definition.auth`**| yes      | Object containing service authentication information |
| **`definition.auth.type`**| yes      | Authentication mechanism (none, basic, digest, oauth, etc.) Complete list can be found on the [IANA website](https://www.iana.org/assignments/http-authschemes/http-authschemes.xhtml) |
| **`definition.auth.user`**| no      | User used for authentication |
| **`definition.auth.password`**| no      | Password used for authentication |
| **`definition.auth.token`**| no      | Authentication token |
| **`definition.endpoints`**| yes      | Array of Endpoint objects including the details of each individual service url |
| **`definition.endpoints.index`**| yes      | Index number of the service endpoint starting from 0  |
| **`definition.endpoints.url`**| yes      | Endpoint URL | 
| **`definition.endpoints.method`**| yes      | HTTP Method (GET, POST, PUT, PATCH, DELETE, HEAD) | 
| **`definition.endpoints.contentTypes`**| yes      | Array of content types supported|

Example of a service:

```json
{  
  "service": [  
    {
      "index": 0,
      "serviceEndpoint": "http://aquarius:5000/api/v1/aquarius/assets/ddo/{did}",
      "type": "metadata",
      "attributes": {  
        "main": {  
          "type": "service",
          "service": {  
            "spec": "https://my.service.inet:8080/spec",
            "specChecksum": "859486596784567856758aaaa",
            "definition": {  
              "auth": {  
                "type": "basic",
                "user": "aitor",
                "password": "1234",
                "token": "89c06eb5a88f4bbbf4ac966d737593b36e61e885"
              },
              "endpoints": [  
                {  
                  "index": 0,
                  "url": "https://my.service.inet:8080/api/v1/weather",
                  "method": "POST",
                  "contentTypes": ["application/json"]
                }
              ]
            }
          }
        }
      }
    }
  ]
}
```

## References

[Schema.org](https://schema.org/) is a collaborative, community activity with a mission to create, maintain, and promote schemas for structured data on the Internet. Data types use the [Schema.org primitive data types](https://schema.org/DataType).

* [Schema.org: DataSet](https://schema.org/Dataset)
* [Schema.org: FileSize](https://schema.org/fileSize)
* [Common license types for datasets](https://help.data.world/hc/en-us/articles/115006114287-Common-license-types-for-datasets)

## Change Process

This document is governed by [OEP 2/COSS](../2/README.md).

## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.
