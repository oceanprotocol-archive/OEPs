```
shortname: 5/AGENT
name: Ocean Agent
type: Standard
status: Raw
editor: Aitor Argomaniz <aitor@oceanprotocol.com>
contributors: Dimitri De Jonghe <dimi@oceanprotocol.com>
```


Table of Contents
=================

   * [Ocean Agent Architecture](#ocean-agent-architecture)
      * [Change Process](#change-process)
      * [Language](#language)
      * [Motivation](#motivation)
      * [High Level Architecture](#high-level-architecture)
      * [Responsibilities](#responsibilities)
      * [Components](#components)
         * [Interfaces](#interfaces)
            * [Communication with Keeper](#communication-with-keeper)
               * [Ocean DB integration](#ocean-db-integration)
            * [Agents P2P communication](#agents-p2p-communication)
            * [Interfaces with external providers](#interfaces-with-external-providers)
      * [Access Control](#access-control)
         * [Authentication](#authentication)
         * [Authorization](#authorization)
      * [PKI](#pki)
         * [Accounts](#accounts)
         * [Wallets](#wallets)
      * [Privacy Management](#privacy-management)
      * [Events Watcher](#events-watcher)
      * [Orchestration Layer](#orchestration-layer)
         * [Data Caching](#data-caching)


<a name="ocean-agent-architecture"></a>
# Ocean Agent Architecture

This document describes the Ocean Agent Architecture. It focus in which are the main responsibilities, functions and components implementing the architecture of the component.

This specification is based on [Ocean Protocol technical whitepaper](https://github.com/oceanprotocol/whitepaper).

This specification is called **AGENT** henceforth.

<a name="change-process"></a>
## Change Process
This document is governed by the [2/COSS](../2/README.md) (COSS).

<a name="language"></a>
## Language
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.

<a name="motivation"></a>
## Motivation

The goal of this document is to describe the responsibilities and architecture of the AGENT node. 
This AGENT node, as part of the Ocean Network, will interact with different components (Keepers and other Agents),
so the intention of this document is to describe the different interaction patterns.
At the same time, the AGENT implements an internal architecture based in different layers. 
This document MUST to provide a common framework and definition used to describe the technical solution to put in place. 

All the different components detailed, SHOULD be used as building blocks, allowing to compose the different scenarios using those.

<a name="high-level-architecture"></a>
## High Level Architecture

This document use as reference and starting point the Architecture defined by the [3/ARCH](../3/README.md) (ARCH).

<a name="responsabilities"></a>
## Responsibilities

AGENT is a thin abstraction layer. The main responsibilities of AGENT are:

* Exposing a common and stable API to the network consumers. 
* Build the authorization & authentication mechanism to identify the user using the client
* Manage the user PKI information
* Compose transactions that are send to the keeper. 
* Orchestrate lower-level Keeper interactions exposing a higher level API
* Subscribe to some Smart Contract events raised by the Keeper and trigger actions responding to that
* Integrate with other Agents, establishing a peer to peer connection
* Integrate with external services or providers (Compute, Data, ..)
* Simple input validation. Throttling and spam prevention is done at the VM validation level in the keeper. 
* Expose some API's providing alternative consumption mechanisms (synchronous/asynchronous)

<a name="components"></a>
## Components

![Ocean Agent](images/ocean-agent.png)

The **Ocean Agent** (aka **AGENT**) is a software application receiving incoming messages (REST, RPC, etc.) related with 
the Ocean Network interactions, and producing some output messages after interact with the **Ocean Keeper components** (aka **KEEPEERS**). 

Independently of the API consumption mechanism, the Ocean Agent is in charge of building the internal 
object models using the incoming messages provided by the **Keeper components interfaces**.
 
This marshaling and un-marshaling operations will allow using a common internal data model across all the application. 

The Agent also will orchestrate the interaction with the Keeper components, allowing to provide a high level view 
from the consumer side, interacting with the decentralized VM and the **Ocean DB**.

<a name="interfaces"></a>
### Interfaces

In charge of receiving the external requests to interact with the system. Initially the Ocean Agent 
will expose a HTTP RESTful API, but is designed to expose the API's or consume requests in different ways.
Because of that the Ocean Agent should expose the API's in different formats allowing integration mechanisms 
that can be used depending on the use case. The initial consumption mechanisms could be:
 
* **Request/Response** - Provided by the RESTful and the RPC interfaces. Those allow a request/response integration. 
The API will expose different HTTP methods implementing the defined actions.

![Request/Response](images/req-response.png)

* **Async Websocket** - Provided by the Websocket interface. Useful when some users need to be subscribed 
to the changes happening in the KEEPER level. For example, if a change in a contract is happening. 

![Websocket](images/websocket.png)

* **Event Driven** - Provided by a Pub/Sub interface. In some scenarios where the execution of an action 
can take some time (more than 2 or seconds) could be recommended to allow async consumption mechanisms. 
This could be an optimal configuration when the Ocean Agent is running in conjunction with a Marketplace.

![Event Driven](images/pub-sub.png)

<a name="communication-with-keeper"></a>
#### Communication with Keeper

The keeper will expose 3 main block of capabilities to the rest of the world:

* **Decentralized VM** - Providing the Smart Contracts implementing the core business logic
* **Ocean DB** - Interfacing with an external and pluggable storage system
* **Worker** - Accepts challenges via p2p commands 

Those capabilities will be integrated from the AGENT using different protocols. 

The Keeper interface module should implement an extensible interfaces system allowing to plug different 
communication protocols to establish the communication between the Ocean Agent and the the Keepers network. 
This component is in charge of the following capabilities:

* Interact with the Keeper components (Decentralized VM, Ocean DB)
* Compose the transactions payload necessary by the Keeper nodes
* Orchestrate the execution of multiple Keeper requests when be necessary

Initially, HTTP RPC is the easiest candidate to integrate in the communication with the Decentralized VM. 

To interact with the Smart Contracts, the AGENT will provide different DTO's. Those will allow to abstract the integration with the contracts in an easier way.

![Keeper Communication](images/orchestration-dtos.png)

The worker nodes will expose a p2p interface supporting some commands allowing to raise proof challenges.

The implementation of this module is highly linked to the Keeper API definition. 

<a name="ocean-db-integration"></a>
##### Ocean DB integration

Some characteristics about the Ocean DB integration:

* Ocean DB as backend is **optional** and **pluggable**. It means the AGENT MUST be able to run without any Ocean DB as backend
* The AGENT will provide the interfaces to implement by different storage implementations to integrate them as backend
* Is not in the AGENT scope to provide the storage interfaces implementations
* Different users or marketplaces can require to integrate different storage systems providing different capabilities, the interface MUST describe the basic behaviour to implement in order to be integrated in the AGENT
* Because the totally flexible and agnostic approach of the AGENT, defining only the interfaces, would be possible to integrate a centralized (Oracle, Kafka, Elastic Search, etc.) or decentralized backend (BigchainDB, IPFS, etc.)
* The backend could be local, distributed or decentralized depending of the system integrated
* User could plug the Ocean DB to provide discovery capabilities integrated as part of the core providing some data consistency (between VM & DB)

![Pluggable Backend](images/pluggable-backend.png)

The AGENT will provide the mechanism to check the config parameters given during the start-up of the process.
Depending of those parameters, if a Backend is provided, the AGENT will invoke the plugin provided implementing the interface in runtime.

Below you can find an example of a configuration file enabling the integration with an external system. 

```properties

# It could be true or false, false by default
agent.backend.enabled=true

agent.backend.implClass=com.example.mydatabase.MyDatabaseImpl

# All the options in the config tree can be passed as parameters to the backend plugin
agent.backend.config.hostname=localhost
agent.backend.config.port=1234
agent.backend.config.user=john
agent.backend.config.password=john
agent.backend.config.xxxx=WhateverNeeded

``` 

And this could be the initial definition of a generic interface:

```java

public abstract class OceanPluginFactory {
    
    public static OceanPluginFactory builder(Config config) {}
    
}

public interface OceanBackendPlugin {
    
    public boolean connect(Config conf);
    
    public boolean disconnect();
    
    public boolean isConnected();
    
    public boolean write(List<T> items);
    
}

```

The Orchestration Layer will be in charge or invoke the optional backend if it's provided, being always the Decentralized VM the main storage and source of truth of the system.

<a name="agents-p2p-communication"></a>
#### Agents P2P communication

This module is in charge of maintain peer to peer communication between Ocean Agents. 
This communication can be used to implement: 
     
* Direct messaging between parties - Allowing for example the agree Contracts terms before to 
formalize the contract or direct sharing of Assets consumptions information between the provider and the consumer. 
* Assets transferring between different Ocean actors - In the actors that are giving access directly 
to some assets without using a third-party provider, would be possible to share directly the Assets between parties.
     
The implementation of the P2P communication is highly related with the existing p2p libraries. 

<a name="interfaces-with-external-providers"></a>
#### Interfaces with external providers

Ocean Agent SHOULD provide a pluggable mechanism allowing to interact with external providers. 
It could be:

* **Computing Providers** - In charge of providing off-chain or on-chain computing services. 
For example Amazon EC2, XAIN, Enigma, etc.
* **Storage or Data Providers** - In charge of providing storage services off-chain or on-chain. 
Like Amazon S3, IPFS, etc.
* **UX/UI Providers** - In charge of providing visual interfacing with the system.

Because is difficult to define upfront the different providers to integrate, it's important 
to implement a pluggable mechanism allowing to extend the systems supported by the system.

![Ocean Agent Plugins System](images/agent-plugins.png)

In the above picture the Storage, Computing and UX/UI interfaces have the responsibility of 
modelling the interaction with the external systems. Having this approach, support additional 
providers would require only the implementation of the communication with the new provider, 
but the integration with the rest of the system should be simpler.

Depending of the implementation of the system, the usage of one plugin or another one, 
could be made by configuration or using dependency injection.

```java
StorageProvider interface {

    bool store(Asset asset);
    Asset retrieve(URL url);
    ..
}
```

<a name="access-control"></a>
## Access Control

![Agent Access Control](images/agent-access-control.png)

Access Control system implements the architecture where external users or applications are 
Authenticated and Authorized in the system, allowing (or denying) the management of the resources.

In general, authentication is the process of validating that somebody really is who he claims to be. 
Authorization refers to rules that determine who is allowed to do what.  

<a name="authentication"></a>
### Authentication

In this Ocean Agent side the authentication layer is very thin, and it's in charge mainly of 
verifying the public key information associated to the transactions.

In all the HTTP API interactions, the component integrating the API SHOULD send his public key as part 
of the HTTP request using the ```Authorization``` HTTP Header 
(see [RFC 7325](https://tools.ietf.org/html/rfc7235#page-7)).

All the requests giving invalid authentication parameters will return a **HTTP 401 Status code: Unauthorized**. 

<a name="authorization"></a>
### Authorization

In the authorization phase it's necessary to validate that user is able to implement a specific action, 
ie. modify the metadata information of a specific asset. To implement this validating it's necessary
 to use the information associated to the ownership of the resources, it's stored on-chain.

![Agent Authorization](images/agent-authorization.png)

The authentication will be implemented in the conjunction between the Access Control layer and the 
Decentralized VM component running in the Keeper side. 

The Access Control layer implement the association between the user information, validated in the 
authentication layer, and the method execution. 
The Decentralized VM component, using the validations implemented in the Smart Contracts, and the 
associating between the resources and the owners or users able to access the resources, 
will implement the validations allowing to authorize the user. It includes to answer the following questions:

* Is the user sending the request the owner of the resource (msg.sender == owner)? 
The ownership of a resource, typically enable to the owner execute the high restricted operations 
related with the resource (like transfer the ownership or updating data).
* Can the user sending the request access (read or write) to the resource? 
The resource can have associated a Access Control List (ACL) defining who can do what. 
* Can the user sending the request to change the ownership of the resource?

<a name="pki"></a>
## PKI

<a name="accounts"></a>
### Accounts

An account is a human-readable identifier (public key) stored on the decentralized VM. 
Every transaction has its permissions evaluated under the configured authority of an account. 
The grants of each account will be validated by the RBAC system. 
The user permissions must be met for a transaction signed under that authority to be considered valid. 
Transactions are signed by utilizing a client that has a loaded and unlocked a wallet. 

Ocean Agent will provide the capabilities to manage the accounts creation. 
It will use the [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) defition 
for the creation of those.

<a name="wallets"></a>
### Wallets

Wallet component is in charge of protects and makes use of your keys. These keys may or may not be 
granted permission to an account authority on the blockchain.
A wallet manages a private/public key pair which is used to cryptographically sign transactions and 
prove ownership on the network. The Ocean Agent will provide the wallet capabilities that allows 
the monetary interactions in the network.

The reference for wallet definitions to be used are:

* [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) - It describes hierarchical deterministic 
wallets (or "HD Wallets"). Those are wallets which can be shared partially or entirely with different 
systems, each with or without the ability to spend coins. 
* [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) - It describes the 
implementation of a mnemonic code or mnemonic sentence, a group of easy to remember words, 
for the generation of deterministic wallets.
* [BIP44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) - Defines a logical 
hierarchy for deterministic wallets based on an algorithm described in BIP32, 
and purpose scheme described in [BIP43](https://github.com/bitcoin/bips/blob/master/bip-0043.mediawiki).

The main methods to be provided are:

* **New wallet** - It creates a new wallet in the system. It is saved in encrypted format, 
passphrase must be provided
* **Update wallet** - It updates an existing wallet. The passphrase must be provided to unlock the 
account and another to save the updated file
* **Import wallet** - Imports an unencrypted private key from a keyfile and creates a new wallet. 
The keyfile should contain an unencrypted private key in hexadecimal format.
* **List wallets** - List the existing wallets in the scope of the Ocean Agent

It's necessary to check about the security limitations of os.urandom, which depends on the version 
of Python, and the operating system. Some implementations rely on it.


<a name="privacy-management"></a>
## Privacy Management

The **AGENT** will implement a Privacy Protocol allowing to negotiate the privacy requirements between parties.
In a non-homogeneous network, different nodes can provide alternative mechanisms (hardware or software) 
implementing some privacy capabilities.

In this scenario of we can assume that different users running Ocean Agents, can require and implement 
different privacy capabilities to negotiate with other agents. It means, each AGENT will define a list 
of the **"privacy systems"** supported. For example:

```
privacy {
	mpc= "sign,encrypt,query"
	tee= "intel,arm"
	zk= "syft,libsnarks"
}
```

Having 2 different AGENT's in a negotiation, during the protocol hand shake, the information about the 
privacy mechanisms will be shared. If they have a common/compatible method, the conversation between 
them could be started.

<a name="events-watcher"></a>
## Events Watcher

An important part of the Smart Contracts implementation in the Decentralized VM is the triggering of Events. 
Those events could expose system notifications when some relevant actions are happening 
(ie. a Contract is signed, an Asset is curated, etc.).

From the AGENT side, the Events Watcher component will be in charge of watching those events 
to trigger some further actions. 

```javascript
var subscription = web3.eth.subscribe('logs', {
    address: '0x123456..',
    topics: ['0x12345...']
}, function(error, result){
    if (!error)
        console.log(log);
});

// unsubscribes the subscription
subscription.unsubscribe(function(error, success){
    if(success)
        console.log('Successfully unsubscribed!');
});
```

The AGENT will provide the interfaces to:

* Subscribe to all the movements related with an Address (FROM or TO) and filtered by TOPIC. It allows:
  - Subscribe to all the events sent by an actor (FROM)
  - Subscribe to all the events sent to a Smart Contract address (TO)
  - The combination of the previous two
  - Filter by TOPIC (specific event)
* Retrieve all the historic transactions related an Address  


<a name="orchestration-layer"></a>
## Orchestration Layer

Using as input the incoming requests and events, the Orchestration layer is in charge on compose 
complex workflows as a result of the interactions of multiple service executions. 
Some scenarios require the execution of multiple steps before completion, for example:

* Contract Settlement - When it's received an API request providing a proof of service, the Ocean Agent requires to execute:
  - Store the Proof of Service information on-chain
  - Check if all the proof of services were provided, in that case update the state of the contract to Settled
  - Notify all the contract parties
* External DB integration - If an external backend is provided, the Orchestration Layer will invoke the method to persist the objects sent to the Decentralized VM.

To implement that, the Orchestration Layer acts as a mediator between different components. 
The implementation of this can be implemented in two possible ways:

* Using a sync orchestration layer, abstracting/encapsulating the execution of multiple components 
using a [Mediator pattern](https://en.wikipedia.org/wiki/Mediator_pattern). 
The mediator can executes one by one all the steps involved in one execution workflow.
* Evolving the Mediator pattern to introduce a pub/sub mechanism. 
In that case, the mediator publish a new event in a specific topic of the events bus. 
Multiple subscribers can listen to that topic implementing the behaviour of the individual phases. 
Those subscribers can emit events to different topics to notify the state of their actions.

<a name="data-caching"></a>
### Data Caching

The AGENT will need to integrate a local CACHE system to coordinate the consistency of the data written to the Decentralized VM and to an optional Ocean DB backend. 
The MAIN OBJECTIVE of the CACHE is to provide a recovery mechanism in case of AGENT failure. This could happens in different situations:

* The AGENT crashes just after to receive an incoming transaction from the API layer and before to be able to invoke the Decentralized VM Smart Contract 
* The Decentralize VM is not available when is invoked by the Orchestration Layer
 
The CACHE should provide the following capabilities:

* It MUST be a Local cache, not exposed in any way to the network
* If AGENT crashes, the AGENT MUST read the state of cache to continue retrying the persistence of the transactions in the different backends
* The state of the cache is INTERNAL to the local node, so MUST NOT be shared with a third party in any moment
* If any error occurs during the persistence of the content, the retries field MUST be increased
* The cache MUST store the representation of the object to be sent to the Decentralized VM as soon as a new registering request be received
* The cache status MUST be UPDATED after the content be stored in the Decentralized VM
* The row representing the object in CACHE MUST be DELETED after the data been stored in the Decentralized VM (and Ocean DB if is provided)
* The CACHE system MUST be thread safe, allowing multiple threads pulling data from the CACHE    

![Cache Interactions](images/cache-sequence.png "Cache Interactions")

In the above image can be viewed the CACHE provides persistence mechanism allowing to work as source of truth during the interaction with the data stores.
At the start of the AGENT, the system can sync the state with the CACHE. In case a previous failure, the CACHE MUST include the pending transactions to be applied. In that case, the AGENT can pickup from the CACHE those and apply the modifications.
After a normal operation, the CACHE MUST delete the completed transaction. 

The cache should store the following information:

| Attribute         | Description|
|:------------------|:----------|
|id                 |Id of the content, in this case obectId|
|type               |Type of the content,It could be "ASSET", "ACTOR", "CONTRACT", etc.|
|status             |Status. Options are: CACHED, STORED_VM, STORED_DB 
|retries            |Number of retries before to remove from the cache. If 0, no limit
|ttl                |Number of seconds after the creationDatetime before to remove from the cache. If 0, no limit 
|creationDatetime   |Creation datetime
|updateDatetime     |Update datetime
|content            |Payload of the content (Json or Avro representation of the Asset)





