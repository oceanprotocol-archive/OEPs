```
shortname: 12/EXEC
name: Execution of Computing Services
type: Standard
status: Raw
editor: Aitor Argomaniz <aitor@oceanprotocol.com>
contributors: Dimitri De Jonghe <dimi@oceanprotocol.com>,
			  Ahmed Ali <ahmed@oceanprotocol.com>

```

Table of Contents
=================

   * [Table of Contents](#table-of-contents)
   * [Execution of Computing Services using Service Agreements](#execution-of-computing-services-using-service-agreements)
      * [Change Process](#change-process)
      * [Language](#language)
      * [Motivation](#motivation)
      * [Actors](#actors)
         * [Technical components](#technical-components)
      * [Flow](#flow)
         * [Workflows](#workflows)
         * [Publishing an Asset including Computing Services](#publishing-an-asset-including-computing-services)
         * [Setting up the Service Execution Agreement](#setting-up-the-service-execution-agreement)
         * [Execution phase](#execution-phase)
      * [Infrastructure Orchestration](#infrastructure-orchestration)



---


# Execution of Computing Services using Service Agreements

This OEP introduces the integration pattern for the use **Service Execution Agreements (SEA)** as contracts between parties interacting in the execution of a Compute Service transaction.
This OEP using the SEA as core element to orchestrate the publishing/execution of this type of computing services.

The intention of this OEP is to describe the flow and integration pattern independently of the Cloud Computing Service.
This could happen using a classical cloud provider like Amazon EC2 or Azure, but also can be used to integrate web3 compute providers like Fitchain or On-Premise infrastructure.

It's out of the scope to detail the Service Execution Agreements implementation. Service Agreements are described as part of the Dev-Ocean repository.

## Change Process

This document is governed by the [2/COSS](../2/README.md) (COSS).


## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.


## Motivation

The main motivations of this OEP are:

* Identify the actors involved
* Detail the main characteristics of this interaction
* Specify the pros and cons of this approach
* Identify the modifications required to integrate in the Ocean stack
* Identify the API methods exposed via the different libraries

## Actors

The different actors interacting in this flow are:

* PROVIDERS - Provide access to the Computing Services
* CONSUMERS - Want to make use of the Computing Services
* MARKETPLACES or DOMAINS - Store the DDO/Metadata related with the Assets/services
* COMPUTE PROVIDER - Cloud or on-premise provider of computation. Typically Amazon, Azure, etc.


### Technical components

The following technical components are involved in an end-to-end publishing and consumption flow:

* [MARKETPLACE](https://github.com/oceanprotocol/pleuston) - Exposes a web interface allowing the users to publish and purchase assets and services associated to those assets. Also facilitates the discovery of those assets.
* SQUID - Library encapsulating the Ocean Protocol business logic. Interacts with all the different components/APIs of the system. Currently it's provided in the following languages:
  - [Squid Javascript](https://github.com/oceanprotocol/squid-js) - Javascript version of Squid to be integrated with Frontend applications.
  - [Squid Python](https://github.com/oceanprotocol/squid-py) - Python version of Squid to be integrated with Backend applications. The primary users are data scientists.
  - [Squid Java](https://github.com/oceanprotocol/squid-java) - Java version of Squid to be integrated with Backend applications. The primary users are data engineers.
* [KEEPER CONTRACTS](https://github.com/oceanprotocol/keeper-contracts) - Provides the Service Execution Agreement (SA) business logic.
* [BRIZO](https://github.com/oceanprotocol/brizo) - Micro-service to be executed by a PROVIDER. It exposes the HTTP REST API permitting access to PUBLISHER Assets or additional services like computation.
* [AQUARIUS](https://github.com/oceanprotocol/aquarius) - Micro-service to be executed by the MARKETPLACES. Facilitates creating, updating, deleting and searching the Asset's metadata registered by the PUBLISHERS. This Metadata, is included as part of a [DDO](../7/README.md), which also includes the Services associated with the Asset (Consumption, Computation, etc.).


## Flow

This section describes the Asset Compute Service flow in detail.
There are some parameters used in this flow:

* **DID** - See [OEP-7](../7/README.md).
* **serviceAgreementId** - Is the unique ID referring to a Service Execution Agreement established between a PUBLISHER and a CONSUMER. The CONSUMER (via Squid) is the one creating this random unique serviceAgreementId.
* **serviceDefinitionId** - Identifies one service in the array of services included in the DDO. It is created by the PUBLISHER (via Squid) upon DDO creation and is associated with different services.
* **templateId** - Identifies a unique Service Agreement template. The Service Agreement is an instance of one existing template. In the scenario described in this OEP, the templateId is `hash(2):ad7c5bef027816a800da1736444fb58a807ef4c9603b7848673f7e3a68eb14a5`

### Workflows

From a high-level point of view, a workflow may be considered a view or representation of a real work.
A workflow consists of an orchestrated and repeatable pattern of activities transformed into processes that transform or process information.

In Ocean we use the concept of workflow to represent a list of tasks to accomplish with the intention of process data.

From a technical point of view, a workflow is a type of Asset (it takes advantage of all the Ocean plumbing of registering, metadata publishing, resolving, etc.). 
The main objective of a workflow is to describe an execution pipeline. A workflow can be splitted in sequential stages, having each stage input, algorithm and output.

In the below example, a workflow is modeled in a JSON document with the following characteristics:

* It has a list of sorted stages by the `stages.index` parameter to be executed sequencially
* Each stage has a list of optional minimum requirements, like the image where it will be executed, cpu required, memory, etc.
* Each stage has an array of sorted input parameters. Each input parameter may be: 
  - A DID (example: `"id": "did:op:12345"`)
  - The output of a previous stage (example: `"previousStage": 0`)
* Each stage has one transformation entry. It includes the id (DID) of the asset in charge of process the input to generate some output
* Each stage includes an entry with some additional output details. This could be a DID or a specific detail about the expected output.

Example of a Workflow:

```json
{
  "type": "Workflow",
  "serviceDefinitionId": "1",
  "stages": [
    {
      "index": 0,
      "stageType": "Filtering",
      "requirements": {
        "image": "did:op:1234",
        "cpu": 2,
        "memory": "8gb",
        "nodes": 1
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
      "output": {} 
    },
    {
      "index": 1,
      "stageType": "Transformation",
      "requirements": {
        "image": "did:op:1234",
        "cpu": 8,
        "memory": "32gb",
        "nodes": 2
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
      "output": {} 
    }

  ]
}  
```

You can find a complete DDO of type workflow in the [ddo.workflow.json example file](ddo.workflow.json).

As a new kind of asset, the workflow details will be persisted inside a DDO in a new service of type "Workflow". Typically an Asset of type workflow, will include in the DDO:

* The Workflow model as part of the DDO.services array
* The Workflow metadata as part of the existing Metadata service

A workflow, as every DDO in Ocean, can be resolved using the assetId (DID).

### Publishing an Asset including Computing Services


1. PUBLISHER generates a DID. See [How to compute a DID](https://github.com/oceanprotocol/OEPs/tree/master/7#how-to-compute-a-did).
1. PUBLISHER creates a DDO including the following information:
   - DID
   - Metadata. It contains the asset name, description, etc. For more details see [OEP-8](../8/README.md).
   - Public key of the PUBLISHER
   - A list of services (Access, etc). For more details see [OEP-11](../11/README.md).

   Each service in the list contains certain information depending on its type.
   Here we document the **Computing** service. The **Access** and **Metadata** services where discussed in the scope of the [OEP-11](../11/README.md).

   A service of type "Computing" contains:

   - Service Definition ID (`serviceDefinitionId`); this helps PUBLISHER find the service definition of a DDO signed by CONSUMER
   - Service Agreement Template ID (`templateId`); points to a deployed on-chain SEA Template Smart Contract. In this case is a template implementing the Computing end to end flow
   - Service endpoint (`serviceEndpoint`); CONSUMERS signing this service send their signatures to this endpoint to request the execution of a workflow

   - A list of condition keys; condition key is the `keccak256` hash of the following:
     * SLA template ID
     * controller contract address (obtained from the solidity contract json file matching the contract name in the SLA condition)
     * controller contract function fingerprint (referred to as function signature or selector)

   - For each condition, a list is required of its parameter values, a timeout, a set of fields determining what conditions depend on other conditions, and a mapping of events emitted by the condition to the off-chain handlers of these events
   - Each event is identified by name. Each event handler is a function from a whitelisted module
   - Service Agreement contract address and the event mapping in the same format as the condition events, for off-chain listeners
   - An integer defining when the agreement is fulfilled in case there are multiple terminal conditions, according to the Service Agreement smart contract

   A service of type "Computing" contains one endpoint:
   - **serviceEndpoint** - A URL to call when the CONSUMER request the execution of a workflow

    An example of a complete DDO can be found [here](./ddo.example.json). Please do note that the condition's order in the DID document should reflect the same order in on-chain service agreement.

1. PUBLISHER publishes the DDO in the Metadata Store (OceanDB) using AQUARIUS. This DDO must include at least one service of type "Computing".

[Here](ddo.computing.json) you have an example of the DDO including a Computing service. Below you can find a small fraction of this:

```json
  "service": [{
    "type": "Computing",
    "serviceDefinitionId": "0",
    "serviceEndpoint": "http://mybrizo.org/api/v1/brizo/services/computing/exec?consumerAddress=${consumerAddress}&serviceAgreementId=${serviceAgreementId}",
    "templateId": "804932804923850985093485039850349850439583409583404534231321131a",
    "provider": {
      "type": "Azure",
      "description": "",
      "environment": {
        "flavour": "tensorflow/tensorflow:latest",
        "typeContainer": "xlsize",
        "cpu": "16",
        "gpu": "0",
        "memory": "128gb",
        "disk": "160gb",
        "maxExecutionTime": 86400
      }
    },

    "serviceAgreementTemplate": {
      "contractName": "EscrowExecComputeTemplate",
      "events": [
        {
          "name": "AgreementCreated",
          "actorType": "consumer",
          "handler": {
            "moduleName": "escrowExecComputeTemplate",
            "functionName": "fulfillLockRewardCondition",
            "version": "0.1"
          }
        }
      ],
      "fulfillmentOrder": [
        "lockReward.fulfill",
        "escrowReward.fulfill"
      ],
      "conditionDependency": {
        "lockReward": [],
        "grantSecretStoreAccess": [],
        "releaseReward": [
          "lockReward",
          "execCompute"
        ]
      },
    "conditions": [ ]
  }]
```

1. PUBLISHER registers the DID, associating the Asset DID to the Aquarius Metadata URL that resolves the DID to a DDO.
To do that, SQUID needs to integrate the `DIDRegistry` contract using the `registerAttribute` method.


![Publishing of a Computing Service](images/computing-setup.png)


### Setting up the Service Execution Agreement

Using only one Squid call `registerAsset(asset_metadata, services_description, publisher_public_key)`, the PUBLISHER should be able to register an Asset including a **Computing** service.
The `services_description` attribute includes the different services (like computing) associated to this asset.

During this phase, through the CONSUMER and the PROVIDER (via BRIZO) negotiation, the Service Execution Agreement (SEA) is created and initialized.

Using Squid calls, a CONSUMER can discover, purchase and use the PROVIDER Computing services.

The complete flow for setting up the SEA is:

1. The CONSUMER chooses a service inside a DDO (the CONSUMER selects a `serviceDefinitionId`).

1. The Service Agreement needs to have an associated unique `serviceAgreementId` that can be generated/provided by the CONSUMER.
In the Smart Contracts, this `serviceAgreementId` will be stored as a `bytes32`. This `serviceAgreementId` is random and is represented by a 64-character hex string (using the characters 0-9 and a-f).
The CONSUMER can generate the `serviceAgreementId` using any kind of implementation providing enough randomness to generate this ID (64-characters hex string).

1. The CONSUMER signs the service details. There is a particular way of building the signature documented elsewhere.
The signature contains `(templateId, conditionKeys, valuesHashList, timeoutValues, serviceAgreementId)`. `serviceAgreementId` is provided by the CONSUMER and has to be globally unique.
  * Each ith item in `values_hash_list` and `timeoutValues` lists corresponds to the ith condition in conditionKeys
  * `values_hash_list`: a hash of the parameters types and values of each condition
  * `timeoutValues`: list of numbers to specify a timeout value for each condition.

It is used to correlate events and to prevent the PUBLISHER from instantiating multiple Service Agreements from a single request.

1. The CONSUMER initialize the SEA on-chain `(did, serviceAgreementId, serviceDefinitionId, signature, consumerAddress, workflowId`).

1. The CONSUMER lock the payment on-chain through the `LockRewardCondition` Smart Contract

1. The PROVIDER via BRIZO receives the `LockReward.Fulfilled` event where he/she is the provider for this agreement

1. The PROVIDER grant the execution permissions for the computation on-chain calling the `execComputeCondition.Fullfill` method  

1. The CONSUMER get the `execComputeCondition.Fullfilled` event. When he/she receives the event, can call the Brizo `serviceEndpoint` url added in the DDO to start the execution of the computation workflow.
   Typically: `HTTP POST /api/v1/brizo/services/computing/exec`

1. BRIZO receives the CONSUMER request, and calls the `checkPermissions` method to validate if the CONSUMER address is granted to execute the service
   If user is granted, BRIZO triggers the Execute Algorithm action in the infrastructure



### Execution phase

During this phase, if and only if the CONSUMER is granted, the CONSUMER can request the start of the Computation in the PUBLISHER infrastructure via BRIZO.

The complete flow for the Execution phase is:

1. BRIZO, after receiving the `execution` request from CONSUMER and validate the permissions using the `checkPermissions` function

1. If the CONSUMER is authorized, BRIZO starts the operations in the INFRASTRUCTURE (cloud or on-premise).

1. BRIZO resolves the DID of the Workflow associated with the Service Agreement. The workflow includes the details of the pipeline to execute, including the different stages, inputs and outputs.

1. For each stage included in the workflow, BRIZO orchestrates the following steps:

   - Resolve the DID of the Algorithm associated with the Workflow 
   - Download the Docker Images from the Docker Registry
   - Copy the algorithm to a volume or bucket
   - Initialize the container/s in the Kubernetes cluster of the provider infrastructure
     This container will have attached 3 different volumes. The first one is for the algorithm, the second is for the input data, the third is for the output data
   - After the container is running, the algorithm is executed. The input and output volumes are passed as the first and second parameter to the algorithm
   - The execution logs are retrieved and stored in the output volume
   - After the algorithm finish, the output should be written in the output volume

1. After finalize the execution of the algorithm, BRIZO request the stop and deletion of the container in the Kubernetes cluster

1. BRIZO retrieve from the INFRASTRUCTURE (if it's available) a receipt demonstrating the execution of the service

1. The new ASSET created after the execution is registered as a new ASSET. It should includes the Metadata of the new Asset created.
   The OWNER of this new asset must be the CONSUMER.

1. The Consumer receives an event including the DID of the new ASSET created

1. BRIZO or any other user may requests the `releasePayment` through the KEEPER. It commit on-chain the HASH of the receipt ticket collected from the INFRASTRUCTURE provider.


![Execution Flow](images/computing-executing.png)


## Infrastructure Orchestration

To facilitate the infrastructure orchestration BRIZO integrates with Kubernetes. 
It allows to abstract the execution of Docker containers with computing services independently of the backend (Amazon, Azure, On-Premise).
To support that BRIZO includes the kubernetes driver allowing to wrap the complete execution including:

- Setting up the cluster
- Creation of volumes
- Starting and stopping the service
- Retrieval of logs


 
 



















