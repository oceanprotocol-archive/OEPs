```
shortname: 6/Invoke
name: API to register & invoke computer services   
type: Standard
status: Raw
editor: Mike Anderson <mike.anderson@dex.sg>
contributors: 
```

<!--ts-->

Table of Contents
=================

   * [Table of Contents](#table-of-contents)
   * [Service Invocation](#service-invocation)
      * [Change Process](#change-process)
      * [Language](#language)
      * [Motivation](#motivation)
      * [Specification](#specification)
         * [Proposed Solution](#proposed-solution)
         * [Registering a Service](#registering-a-service)
         * [Retrieve metadata of an Service](#retrieve-metadata-of-a-service)
         * [Updating Service Metadata](#updating-service-metadata)
         * [Retiring a Service](#retiring-a-service)
         * [Invoking a Service](#invoking-a-service)
         * [Providing Service proof](#proving-a-service)
         * [Consuming Service results](#consuming-service-results)
         * [Targeted Release](#targeted-release)
      * [Copyright Waiver](#copyright-waiver)

      
<!--te-->

<a name="service-invocation"></a>
# Service Invocation

The Service Invocation API (**OAR**) is a specification for the Ocean Protocol to register and invoke computational services.

Compute services are defined as services available on the Ocean Network that

* Can be invoked by any Ocean Agent connected to the Ocean Network (subject to access restrictions)
* May accept one or more Input parameters (which will typically be data assets to be used or algorithms to be run)
* Typically produce one or more Outputs (which will typically be references to generated data assets)
* Support the provision of proofs by service providers upon service completion (after which tokens in escrow may be released) 

This OEP does not prescribe the exact type of compute services offered. It is open to service provider implementations to define there, providing that they conform with this API specification
This OEP does not cover service discovery.

This specification is based on [Ocean Protocol technical whitepaper](https://github.com/oceanprotocol/whitepaper), [3/ARCH](../3/README.md), [4/KEEPER](../4/README.md) and [5/AGENT](../5/README.md).


<a name="change-process"></a>
## Change Process
This document is governed by the [2/COSS](../2/README.md) (COSS).

<a name="language"></a>
## Language
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.

<a name="motivation"></a>
## Motivation

Ocean network aims to power marketplaces for relevant AI-related data services.
There is a need for a standardised interface for the invocation of compute services

Requirements are:

* ASSETS are DATA objects describing RESOURCES under control of a PUBLISHER
* SERVICES are compute services according to the protocol defioned in this OEP
* PROVIDERS register and offer SERVICES 
* PROVIDERS may publish SERVICE METADATA relating to the service offered
* CONSUMER can identify services with a unique ID on the Ocean Network 
* CONSUMER can invoke services (subject to access requirements) 
* SERICES may require INPUTS 
* SERICES may produce OUTPUTS
* INPUT ASSETS must be available for the service provider to consume
* PROVIDER provides SERVICE and PROOF VERIFIER validates PROOF
* OUTPUT ASSETS, if created, must be identified and communicated to the CONSUMER 
  
<a name="specification"></a>
## Specification 

The **Serice Metadata** information should be managed using an API on the Ocean Agent. 
As general rule, only the INDISPENSABLE information to run the Smart Contracts MUST be stored in the Decentralized VM

This API should exposes the following capabilities in the Ocean Agent:

* Registering a new Service
* Retrieve metadata information of an Service
* Update the metadata of an existing Service 
* Retire a Service

The following restrictions apply during the design/implementation of this OEP:

* The SERVICES registered in the system MUST be associated to the PROVIDER registering the services
* Basic information about the SERVICES (ids, pricing, reference to service endpoints) MUST be stored in the Decentralized VM  
* AGENT MUST NOT store or rely on any other information about the Services during this process
