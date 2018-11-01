```
shortname: 19/ENDPOINTS
name: Ocean Standard Endpoints
type: Standard
status: Raw
editor: Mike Anderson <mike.anderson@dex.sg>
contributors: 
```

**Table of Contents**

<!--ts-->

   * [Ocean Standard Endpoints](#ocean-standard-endpoints)
      * [Change Process](#change-process)
      * [Language](#language)
      * [Motivation](#motivation)
      * [Specification](#specification)
         * [Endpoint Definition](#endpoint-definition)
         * [Current Endpoint Types](#current-endpoint-types)
      * [Changes Required](#changes-required)
      * [References](#references)

<!--te-->

# Ocean Standard Endpoints

This OEP defines how API endpoints should exposed by Ocean Agents.

In order to enable clients to locate specfic APIs, we must define standard endpoints.
These endpoints are optional, but required if an Identity wishes their endpoints to be consumed
by standard Ocean client tools.

Endpoints exposed should be declared in the relevant DDO of an Actor who wished to offer these endpoints.

## Change Process

This document is governed by the [2/COSS](../2/README.md) (COSS).


## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.


## Motivation

The main motivations of this OEP are:

* Define a standard set of endpoints for Ocean Agent APIs
* Describe how these endpoints are specified within a DDO

## Specification

Requirements are:

* Each Ocean API must be allocated a unique endpoint type
* Each DDO may specify one or more services as per the W3C DID spec
* If the service represents an Ocean API, the "type" of the service must be the endpoint type
* The endpoint type should include a version number 


## Proposed Solution

### Endpoint definition

Endpoints are specified in the following form in the DDO:

```json
{
  "service": [{
    "type": "Ocean.Meta.v1",
    "serviceEndpoint": "https://mobi.com/meta"
  }]
}
```

### Current endpoint types

Endpoint type          |   Description
-----------------------|----------------------
Ocean.Meta.v1          | Endpoint for the Meta Agent API v1 (OEP15 - TBC)
Ocean.Market.v1        | Endpoint for the Market Agent API
Ocean.Storage.v1       | Endpoint for a generalised storage API
Ocean.Invoke.v1        | Endpoint for an invokable service API


## Changes Required

The list of changes to apply in the proposed solution are:

* Modify client libraries to look up the relevant endpoint from the DDO if available
* Modify registration code for Agents to specify the correct endpoint types

## References

* https://w3c-ccg.github.io/did-spec/
