```
shortname: 12/EXEC
name: Execution of Compute Services
type: Standard
status: Raw
editor: Ahmed Ali <ahmed@oceanprotocol.com>
contributors:
        Aitor Argomaniz <aitor@oceanprotocol.com>,
        Samer Sallam <samer@oceanprotocol.com>,
        Javier Cortejoso <javier@oceanprotocol.com>,
        Enrique Ruiz <enrique@oceanprotocol.com>, 
        Troy <troy@oceanprotocol.com>,
        Dimitri De Jonghe <dimi@oceanprotocol.com>,
        Ahmed Ali <ahmed@oceanprotocol.com>,
        Jose Pablo Fernandez <jose@oceanprotocol.com>,
        Alex Coseru <alex@oceanprotocol.com>

```

Table of Contents
=================

  
  * [Introduction](#introduction)
  * [Change Process](#change-process)
  * [Language](#language)
  * [Motivation](#motivation)
  * [Actors](#actors)
  * [Technical components](#technical-components)
  * [Compute Flow](#compute-flow)
 	* [Requirements](#requirements)
	* [Publishing an Asset including Compute Services](#publishing-an-asset-including-compute-services)
	* [Consuming a compute service](#consuming-a-compute-service)
        * [Setting up the Service Execution Agreement](#setting-up-the-service-execution-agreement)
            * [Creating the agreement on-chain](#creating-the-agreement-on-chain)
            * [Payment and service approval](#payment-and-service-approval)
        * [Running the compute job](#running-the-compute-job)
            * [Infrastructure Orchestration](#infrastructure-orchestration)
            * [Infrastructure Operator](#infrastructure-operator)
                * [Network isolation](#network-isolation)
                * [Volumes](#volumes)
                * [Compute image details](#compute-image-details)


---


# Introduction

This OEP introduces the integration pattern for the usage of **Service Execution Agreements (SEA)** 
(also called Service Agreements or Agreements) as contracts between parties interacting in the execution of 
a Compute Service transaction. This OEP using the SEA as core element, orchestrates the publishing/execution 
of this type of compute services.

The intention of this OEP is to describe the flow and integration pattern independently of the infrastructure Cloud Compute Service.
This OEP MUST be valid for integrating classical infrastructure cloud providers like Amazon EC2 or Azure, 
but also can be used to integrate web3 compute providers or On-Premise infrastructure.

It's out of the scope to detail the Service Execution Agreements implementation. 
Service Agreements are described as part of the [Dev-Ocean repository](https://github.com/oceanprotocol/dev-ocean).

>**Disclaimer**: The current focus of this OEP is to bring the compute (or algorithm) to data 
which assumes that the DATA PROVIDER trusts the COMPUTE PROVIDER and hence COMPUTE PROVIDER has 
access to data in case they are not the same entity. For this OEP, we will assume that 
DATA PROVIDER and COMPUTE PROVIDER are the same entity.

# Change Process

This document is governed by the [2/COSS](../2/README.md) (COSS).


# Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", 
"NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described 
in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \
[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.


# Motivation

The main motivations of this OEP are:

* Identify the actors involved in the Compute service
* Define the execution procedures of this interaction
* Identify the modifications required to integrate in the Ocean stack
* Identify the API methods exposed via the different libraries

# Actors

The different actors interacting in this flow are:

* DATA PROVIDERS - one who provides access to Data
* CONSUMERS - one who wants to use Compute Services
* MARKETPLACES OR DAPPS- one who facilitates this data and compute exchange and stores the DDO/Metadata related with the Assets/services
* COMPUTE PROVIDERS - one who provides compute services via cloud or on-premise infrastructure services.



# Technical components

The following technical components are involved in an end-to-end publishing and consumption flow:

* [MARKETPLACE](https://github.com/oceanprotocol/commons) - 
Exposes a web interface for asset discovery and allowing users to publish / consume assets and data related services such as compute.
* SQUID - Library encapsulating the Ocean Protocol business logic. 
Interacts with all the different components/APIs of the system. Currently it's provided in the following languages:
  - [Squid Javascript](https://github.com/oceanprotocol/squid-js) - Javascript version of Squid to be integrated with Frontend applications.
  - [Squid Python](https://github.com/oceanprotocol/squid-py) - Python version of Squid to be integrated with Backend applications. The primary users are data scientists.
  - [Squid Java](https://github.com/oceanprotocol/squid-java) - Java version of Squid to be integrated with Backend applications. The primary users are data engineers.
* [KEEPER CONTRACTS](https://github.com/oceanprotocol/keeper-contracts) - Provides the Service Execution Agreement (SEA) business logic.
* [BRIZO or GATEWAY](https://github.com/oceanprotocol/brizo) - Micro-service to be executed by a PROVIDER. 
It exposes the HTTP REST API permitting access to data assets or additional services like running computation.
* [AQUARIUS](https://github.com/oceanprotocol/aquarius) - Micro-service to be executed by the MARKETPLACE. 
Facilitates creating, updating, deleting and searching the Asset's metadata registered by the asset PROVIDER. 
This Metadata, is included as part of a [DDO](../7/README.md), which also includes the Services associated with 
the Asset (Download, Computation, etc.).
* [OPERATOR SERVICE](https://github.com/oceanprotocol/operator-service) - a micro-service in charge of managing 
the workflow executing requests. Typically the Operator Service is integrated from the Brizo proxy, but can be called independently of it.
* [OPERATOR ENGINE](https://github.com/oceanprotocol/operator-engine) - a backend agent in charge of orchestrating
the compute infrastructure using Kubernetes as backend. Typically the Operator Engine retrieve the Workflows 
created by the OPERATOR SERVICE, in Kubernetes and manage the infrastructure necessary to complete the execution of the compute workflows.


# Compute Flow

This section describes the Asset Compute Service flow in detail.
Below are some parameters and their significance used in this flow:

* **DID** - See [OEP-7](../7/README.md).
* **serviceAgreementId** - Is the unique ID referring to a Service Execution Agreement established between a PROVIDER and a CONSUMER. The CONSUMER (via Squid) is the one creating this random unique serviceAgreementId.
* **index** - Identifies one service in the array of services included in the DDO. It is created by the PROVIDER (via Squid) upon DDO creation and is associated with different services.
* **templateId** - Identifies a unique Service Agreement template. The Service Agreement is an instance of one existing template. Please refer to this [documentation](https://github.com/oceanprotocol/keeper-contracts/blob/develop/doc/TEMPLATE_LIFE_CYCLE.md) for more info.

  

## Requirements

* A COMPUTE PROVIDER or PROVIDER define the conditions that a Compute service supports. It includes:
  - What kind of image (Docker container) can be deployed in the infrastructure
  - What are the infrastructure resources available (CPU, memory, storage)  
  - What is the price of using the infrastructure resources
* A COMPUTE PROVIDER defines a Compute Service in the scope of the Asset (DID/DDO) of the dataset that can be analyzed or used for model training
* A CONSUMER defines the algorithm and its requirements to run in the compute service
* A CONSUMER can purchase a service given by a PROVIDER and execute multiple times until the agreement expires
* A CONSUMER could purchase a service and execute later, the purchase MUST be totally decoupled of execution
* The previous two points could support to buy a compute service once and execute for example the service every night at 3 am



## Publishing an Asset including Compute Services
The process of publishing an asset is described in [OEP-11](https://github.com/oceanprotocol/OEPs/tree/master/11/v0.2#publishing) 
which includes the addition of a service of type "access". To enable the compute service, add a service of type "compute` to the 
asset's list of services (under the "service" attribute in the DDO document). The asset type in the metadata section is still 
set to **dataset**.

Note the following attributes:
* type: The type of service, must be set to `compute` 
* servicEndpoint: The URL to use to access the compute service (i.e. to run a remote compute job)
* price: An integer, the price to pay for this service in `OCEAN` ERC20 tokens expressed in `vodka` units; where 1 `OCEAN` token = 10 ** 18 `vodka`
* timeout: Expiry of this service agreement. The consumer can use the compute service until the agreement has expired 
* serviceAgreementTemplate: defines the parameters to use in the `EscrowComputeExecutionTemplate` agreement
* privacy : allows fine tunning of privacy:
  * allowRawAlgorithm : if Raw Algorithms are allowed or not
  * allowNetworkAccess : allow network access in Algorithm pod
  * trustedAlgorithms : list of allowed DIDs of type algorithm. **If the list is empty, then any Algorithm DID is allowed.**

The following is an example of a "compute" type service -

```
{
      "type": "compute",
      "index": 2,
      "serviceEndpoint": "http://mybrizo.org/api/v1/brizo/services/compute",
      "attributes": {
        "main": {
          "name": "dataAssetComputingServiceAgreement",
          "creator": "0x00Bd138aBD70e2F00903268F3Db08f2D25677C9e",
          "datePublished": "2019-04-09T19:02:11Z",
          "price": "10",
          "timeout": 86400,
          "privacy":{
            "allowRawAlgorithm":true,
            "trustedAlgorithms": [],
            "allowNetworkAccess":true
          },
          "provider": {
            "type": "Azure",
            "description": "",
            "environment": {
              "cluster": {
                "type": "Kubernetes",
                "url": "http://10.0.0.17/xxx"
              },
              "supportedContainers": [
                {
                  "image": "tensorflow/tensorflow",
                  "tag": "latest",
                  "checksum": "sha256:cb57ecfa6ebbefd8ffc7f75c0f00e57a7fa739578a429b6f72a0df19315deadc"
                }
              ],
              "supportedServers": [
                {
                  "serverId": "1",
                  "serverType": "xlsize",
                  "price": "50",
                  "cpu": "16",
                  "gpu": "0",
                  "memory": "24gb",
                  "disk": "160gb",
                  "maxExecutionTime": 86400
                }
              ]
            }
          }
        },
        "additionalInformation": {},
        "serviceAgreementTemplate": {
          "contractName": "EscrowComputeExecutionTemplate",
          "events": [
            {
              "name": "AgreementActorAdded",
              "actorType": "provider",
              "handler": {
                "moduleName": "",
                "functionName": "fulfillLockRewardCondition",
                "version": "0.1"
              }
            }
          ],
          "fulfillmentOrder": [
            "lockReward.fulfill",
            "computeExecution.fulfill",
            "escrowReward.fulfill"
          ],
          "conditionDependency": {
            ...
          },
          "conditions": [
            {
              "name": "lockReward",
              ...
            },
            {
              "name": "computeExecution",
              ...
            },
            {
              "name": "escrowReward",
              ...
            }
          ]
        }
      }
    }
```

## Consuming a compute service
A user can buy the compute service by creating an SEA on-chain according the service agreemnt 
template found in the asset's `compute` service. This involves submitting the tokens payment 
and waiting for an approval by the provider proxy.

Once the provider approval is submitted on-chain, the user can use the serviceEndpoint to 
send start compute requests.
 
### Setting up the Service Execution Agreement
The compute to data use case follows the same pattern of agreement initialization like that of dataset publish and consume,
by pointing to the DID, consumer address, provider address and the agreement template (set of 
predefined conditions, and actor types).

#### Creating the agreement on-chain
To create new agreement, the consumer should follow the below sequence diagram:
![](images/2_createAgreement.png)

For a given agreement, consumers are allowed to create ***N*** compute jobs with a time limit based on the agreement conditions.

#### Payment and service approval
The compute to data agreement uses `EscrowComputeExecutionTemplate` which is defined by three conditions:

- **LockRewardCondition**: allows CONSUMER to lock ERC20 tokens/OCEAN tokens in escrow contract.
- **ComputeExecutionCondition**: allows COMPUTE PROVIDER to confirm and fulfill the 
computation request thus approving the agreement.
- **Release/RefundRewardCondition**: allows COMPUTE PROVIDER to release the payment after 
`ComputeExecutionCondition` is fulfilled or allows CONSUMER to withdraw their payment after 
`ComputeExecutionCondition` timeout if the computation service was not confirmed.

![](images/3_executeSEAPart1.png)

### Running the compute job

In this part, the trigger of the agreement execution goes from on-chain (the keeper) 
to the provider service endpoint [Brizo](https://github.com/oceanprotocol/brizo) in order 
to handle the compute job by calling the operator service. Moreover, 
[Brizo](https://github.com/oceanprotocol/brizo) exposes the same endpoints of the operator 
service which will be discussed in the section below.

#### Infrastructure Orchestration
The infrastructure is orchestrated by [operator service](https://github.com/oceanprotocol/operator-service) 
and [operator-engine](https://github.com/oceanprotocol/operator-engine) which configures the pods (workers), and manages the life cycle of compute 
jobs. The APIs are as follows:

- **start**: starts a new job within the context of the new/current agreement.
An algorithm script/code can be included in the `start` compute payload. Alternatively, a pre-published 
algorithm DID can be used, the provider should be able to get the associated DDO document and setup the 
algorithm for running in the compute job
- **stop**: stop running job. This requires valid agreement Id, job id, and job ownership proof (signature).
- **status**: For a given agreement Id, and (job id -- optional) returns job status(es) and includes results URLs. Status code description below.

***Starting new compute job***

![](images/4_Starting_New_Compute_Job.png)


***Stop Existing Job***

![](images/5_Stop_Compute_Job.png)

***Get Job Status and result***

![](images/6_Status_of_Compute_Job.png)

The following table lists all the possible status codes for a compute job

| status    | Description                    |
|-----------|--------------------------------|
|  10       | Job started                    |
|  20       | Configuring volumes            |
|  30       | Provisioning success           |
|  31       | Data provisioning failed       |
|  32       | Algorithm provisioning failed  |
|  40       | Running algorithm              |
|  50       | Filtering results              |
|  60       | Publishing results             |
|  70       | Job completed                  |

The status response is a list of matching jobIds and looks like this:
```json
[
      {
        "owner":"0x1111",
        "agreementId":"0x2222",
        "jobId":"3333",
        "dateCreated":"2020-10-01T01:00:00Z",
        "dateFinished":"2020-10-01T01:00:00Z",
        "status": 70,
        "statusText":"Job completed",
        "algorithmLogUrl":"http://example.net/logs/algo.log",
        "resultsUrls":[
            "http://example.net/logs/output/0",
            "http://example.net/logs/output/1"
         ],
         "resultsDid":"did:op:87bdaabb33354d2eb014af5091c604fb4b0f67dc6cca4d18a96547bffdc27bcf"
       }
]
```

Once the compute job is completed, the results can be accessed using either the `resultsUrls` 
or the `resultsDid` (if the consumer requested publishing the results as a new asset)

For more details, please refer to [operator service APIs documentation](https://github.com/oceanprotocol/operator-service/blob/develop/API.md)

#### Infrastructure Operator

The PUBLISHER of computation services is in charge of defining the 
requirements to allow the execution of algorithms on the data assets.
  
[BRIZO](https://github.com/oceanprotocol/brizo) is in charge of setting up the runtime 
environment speaking with the infrastructure provider via Kubernetes.

##### Network isolation

The runtime environment doesn't need to have network connectivity to external networks to be executed. 
To avoid sending the internal information about the data, it's recommended to restrict the output connectivity. 
   

##### Volumes

The input assets will be added to the runtime environment as **read only** volumes. 
The path to the inputs folder where the volumes are mounted will be given to 
the algorithm as an environment variable `INPUTS`. The inputs folder will contain a 
folder for each asset/dataset and named using the asset DID (only one is supported 
for now). The algorithm can access the input folder names using the list of DIDs 
defined in the environment variable `DIDS`.

The new derived Asset generated as a result of the execution of the algorithm MUST 
be written in the output volume which is also declared in the environment variable `OUTPUTS`. 
There is also the logs folder defined in the environment variable `LOGS`.

The pods will be **destroyed** after the execution, so only the data stored in 
the **output** or **logs** volumes should be used.

| Type       | Permissions  | ENV Parameter  | Default Value  | Comment            |
|------------|--------------|----------------|----------------|--------------------|
| Input      | Read         | INPUTS         | /data/inputs   |                    |
| Output     | Read, Write  | OUTPUTS        | /data/outputs  |                    |
| Logs       | Read, Write  | LOGS           | /data/logs     |                    |
| Input DIDS |  -           | DIDS           | []             | List of input DIDS |


##### Compute image details

Every algorithm has some required attributes which can be defined in an algorithmMeta object 
to include in the payload. This can also be obtained from the algorithm DDO when using a 
published algorithm Asset.
```
          "container": {
            "image": "node",
            "tag": "10",
            "entrypoint": "node $ALGO"
          }
```

`image`: A docker image to run the algorithm in
`tag`: The docker image tag/version
`entrypoint`:  Contains a macro ($ALGO) that gets replaced in the compute 
server with the actual location of the algorithm (usually /data/transformations/algorithm).

So, if you want to run a python script, you can have the following:

```
          "container": {
            "image": "python",
            "tag": "3.7",
            "entrypoint": "python3.7 $ALGO"
          }

```

or a php script

```
algorithm": {
          "container": {
            "image": "php",
            "tag": "cli",
            "entrypoint": "/usr/bin/php $ALGO"
          }
```
Or a simple bash script:
```
          "container": {
            "image": "ubuntu",
            "tag": "18.04",
            "entrypoint": "$ALGO"
          }
```
