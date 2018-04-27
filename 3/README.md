```
shortname: 3/ARCH
name: Ocean Protocol Architecture
type: Standard
status: Raw
editor: Aitor Argomaniz <aitor@oceanprotocol.com>
contributors: Dimitri De Jonghe <dimi@oceanprotocol.com>
```

# Ocean Protocol Architecture

This document describes the Ocean Architecture. 
It focus in which are the main responsibilities, functions and components implementing the architecture.

This specification is based on [Ocean Protocol technical whitepaper](https://github.com/oceanprotocol/whitepaper).

This specification is called **ARCH** henceforth.

## Change Process
This document is governed by the [2/COSS](../2/README.md) (COSS).

## Language
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.

## Motivation

Ocean Protocol connects data service PROVIDERS with CONSUMERS using 
contractual agreements to ensure payments, audit trails and privacy.

The ARCH covers a conceptual framework for provisioning contractual CONNECTIONS,
an overview of the Ocean Network architecture and it's building blocks,
as well as a suite of protocols that implements the design. 

This ARCH is heavily inspired by the Internet architecture described in 
[RFC 1122](https://tools.ietf.org/html/rfc1122), 
[RFC 1123](https://tools.ietf.org/html/rfc1123) and 
[RFC 1009](https://tools.ietf.org/html/rfc1009) and the 
[Interledger Protocol (ILP)](https://github.com/interledger/rfcs/blob/master/0001-interledger-architecture/0001-interledger-architecture.md#interledger-architecture).

## Design Requirements

The following design requirements should guide the development of the Ocean Protocol:

* MUST use an open protocol, accessible to clients using any technology stack
* MUST be decentralized in ownership of vision, mission, governance, implementation and usage
* MUST support data services of all types, including:
  - Data assets
  - Data operations including:
    - Data transformation operations
    - Machine learning operations
    - Data interaction and visualisation libraries
* MUST support off-chain operations for storage and compute
* MUST be designed around self-sovereignty in terms of ownership, control and exposure of identities and data services. 
* MUST support arbitrary levels of privacy comfort and regulation (data escapes, right to be forgotten, anonymity, zero-knowledge, etc)
* MUST enable interoperability (i.e. sharing of data assets and services) between multiple marketplace implementations and multiple technology stacks
* MUST store a record of all asset provenance
* MUST allow metadata governance (as defined by marketplaces and/or domain specific use cases)
* MUST support arbitrary data formats (e.g. as specified by MIME type)
* MUST support both free and priced data assets / services
* MUST allow marketplaces to implement custom asset pricing logic (for priced services)
* MUST allow marketplaces to implement custom access control logic (e.g. whitelisting for consumers)
* MUST support the IP management. During the creation of a new Asset or Service, the ownership rights should be registered. During the consumption agreement, this right should be registered.
* MUST give Ocean agents (including but not limited to Marketplaces) an efficient way to access the full blockchain history so that can query or index a copy of the blockchain as necessary
* SHOULD minimise latency for user transactions


## Ocean Protocol Model

This document MUST to provide a common framework used describing 
the technical solution to put in place. 

All the different components described SHOULD be used as building blocks, 
allowing to compose the different scenarios using them.

The ARCH is based in a **layered protocol** stack to set up CONNECTIONS between AGENTS using CONTRACTS.

![Combination of Building Blocks](images/ocean-model.png)

A CONTRACT is the interface between AGENTS and is validated by decentralized consensus at the KEEPER level.
A CONNECTOR is either another Ocean KEEPER or can bridge the replicated state and payments between other consensus networks.

One of the main responsibilities of a CONTRACT between AGENTS is to lock up funds 
and have means to resolve the transfer of funds (for example by verification of CONNECTION-related cryptographic proofs). 
The contract is generic and sent as a payload of a transaction (TX) to the consensus engine.

The interaction between the consensus layer and the AGENTS is driven by transactions:

* As a means of token transfer
* For deploying contracts and registering metadata
* For calling contracts with specific input variables and hence updating the contract state.

It's the responsibility of the AGENTS to construct valid transactions and contracts. 
Otherwise they won't be accepted by the consensus layer.

We foresee a few ways to set up contracts:

* Facilitation by a marketplace
* Peer-to-peer between AGENTS
* Open ended on-chain. ie send a valid TX to the contract address and gain service access

In the remainder of this document we refer to:
- AGENT: client-side software responsible for constructing transactions.
Multiple roles can be foreseen such as CONSUMERS, PROVIDERS, MARKETPLACES, PUBLISHERS, CURATORS and VERIFIERS.
- KEEPER: consensus-side software responsible for validating transactions, 
smart contracts execution, consensus algorithm coordination and decentralized storage.

Below is a diagram that depicts various AGENT roles as well as their interactions.

It should be noted that a minimum viable Ocean network only requires KEEPERS, CONSUMERS and combined PUBLISHER-PROVIDERS.

![Network Architecture with Actors](images/network-arch-with-actors.png)

## Components

The Ocean ARCH is composed of independent components:

![Building Blocks](images/building-blocks.png)

### Keepers

Implements the Ocean Protocol and all the business logic embedded in validation and smart contracts. 
A full node would be running three distinct processes:

#### Ocean VM

A blockchain component with smart contract abilities:

* Interacts with the AGENTS through transactions. 
* Executes the Smart Contracts. The VM serves as a deterministic state transition mechanism.
* Stores transactions, blocks and contract state - such as bytecode, proofs, variables - in a Directed Acyclic Graph (DAG). 
* Is a validation engine for Unspent Transaction Outputs (UTXO) and contract logic.
* Is [power-fault tolerant](https://filecoin.io/power-fault-tolerance.pdf) (PFT) 

#### Ocean DB

A decentralized database with the following capabilities:

* Interacts with the AGENTS through transactions. 
* Stores metadata about assets and actors in a decentralized database. 
* Has a query layer and an indexing scheme
* Is Byzantine fault tolerant (BFT)

#### Ocean Worker

A work-dispatch engine with the following capabilities:

* Interacts with the AGENTS through transactions.
* Performs compute intensive jobs such as mining and proofs validation
* Can challenge AGENTS to provide succinct proofs
* Is fault tolerant (FT)


### Ocean Agent

Thin abstraction layer in charge of exposing a common and stable API to the network consumers. 
The client outputs transactions that are send to the keeper. 
It could provide verification, privacy & multicast capabilities. 
Throttling and spam prevention is done at the VM validation level in the keeper. 
TX's have fees attached, could have some Client-side PoW and invalid tx's won't get replicated by honest nodes.

## Components Interaction

An Ocean Node can run all of those processes or any combination of them. 
It will allow to have specialized nodes, depending of the requirements of the users running those nodes.

![Combination of Building Blocks](images/building-blocks-combination.png)

In the above picture you can see multiple set up combinations of an Ocean Node. 
The first one (top left), shows an Ocean Node running an Agent and a Full Keeper (Decentralized VM, Database and Worker). 
Other scenarios could require specialized deployments to run Worker nodes, Agents, Database nodes, etc.

## Ocean Protocol Suite

The Ocean protocol stack has 7 layers, divided between AGENT and KEEPER.

![Protocol Stack](images/protocol-stack.png)

### Application Layer

The application layer describes the AGENT interface with the outside world - ie. the data service tools.
Protocols at this level are responsible for:

1. Registry and discovery of services and agents
2. Contract selection and details negotiation
3. (Remote) communication and integration with service interfaces, tools and libraries


As an example, consider following data delivery scenario:

- A data PROVIDER publishes a datum ASSET offering together 
with CONNECTION details such as authorization and encryption.

- A data CONSUMER discovers the offering through a user interface.

- Both parties SHOULD negotiate and agree on CONTRACT and CONNECTION details.

- Once the CONTRACT is deployed and signed, the data CONSUMER MUST ACCESS 
the datum using the CONNECTION details. 
The data PROVIDER MAY listen to state changes of the CONTRACT 
and perform callback functions for CONNECTION management and responding 
to cryptographic CHALLENGES.

- At any point the CONTRACT resolution is triggered and MAY revoke ACCESS.

The high level description as well as the  developer and user experience for this scenario
SHALL be implemented at this level. 

Details of the application layer are discussed in [#TODO:OEP-PLUGIN](../<PLUGIN>/README.md) 

### Privacy Layer

The privacy layer SHALL take care of end-to-end privacy for messages traveling over a CONNECTION between AGENTS.
Various protocols are available to enhance privacy of data services such as encryption, 
homomorphic encryption (HE), trusted execution environments, on-premise, multiparty (MPC) and zero-knowledge computation (ZK).

In this layer both AGENTS MAY negotiate privacy details for the CONNECTION.

Due to limited capabilities of privacy suites like MPC, HE and ZK, 
not all service capabilities at the application level WILL supported.

Details of the privacy layer are discussed in [#TODO:OEP-PRIVACY](../<PRIVACY>/README.md) 

### Access Layer

Service CONTRACTs in the Ocean network are the basis for access permissions between AGENTS.
The access layer foresees CONTRACT-based authentication and authorization for end-to-end CONNECTIONS.

The way access can be granted depends heavily on the type of data service and MAY include
signed tokens, signed URLs, on-chain role-based access control (RBAC), OAuth and so forth.

Details of the access layer are discussed in [#TODO:OEP-ACCESS](../<ACCESS>/README.md) 

### Transport Layer

Transport layer protocols foresee the coordination of a service CONTRACT between all involved AGENTS.

Example contract protocols include peer-to-peer escrow contracts, marketplace-based contracts, 
as well as CONTRACT resolution mechanisms such as judging and verification.

This layer SHOULD also listen to events generated by state changes on the keeper 
and provide necessary callbacks and hooks. 

Details of the transport layer are discussed in [#TODO:OEP-TRANSPORT](../<TRANSPORT>/README.md) 

### Bridge Layer

The bridge layer is responsible for forwarding packets between AGENTS. There is only one protocol on this layer.

The bridge protocol defines standard address and packet formats that instruct CONNECTORS where to forward a packet.
Bridge addresses provide a universal way to address AGENTS, KEEPERS and CONNECTORS.

Details of the bridge layer are discussed in [#TODO:OEP-BRIDGE](../<BRIDGE>/README.md) 

### Validation Layer

- plugin system to convert Bridge protocol packages to ledger-specific contracts and transactions

Details of the access layer are discussed in [#TODO:OEP-ACCESS](../<ACCESS>/README.md) 

### Consensus Layer

- decentralized coordination between nodes, gossip protocols

Details of the access layer are discussed in [#TODO:OEP-ACCESS](../<ACCESS>/README.md) 
