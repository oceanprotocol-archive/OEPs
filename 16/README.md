# OEP-16: Network Reward Distribution

```
shortname: 16/REWARD
name: Network Reward Distribution
type: Standard
status: Raw
version: 0.1
editor: Alex Coseru <alex@oceanprotocol.com> 
        Fang Gong <fang@oceanprotocol.com>
        Ahmed Ali <ahmed@oceanprotocol.com>,
        Sebastian Gerske <sebastian@oceanprotocol.com>
```


## Abstract

It is critical for Ocean network to motivate data providers to add more valuable and releavent data commons to our network. As such, the total value of the network as well as Ocean token price can increase over the time. To do so, Ocean network has reserved a partial of total token supply as the network rewards that will be distributed to providers of data commons in the Ocean network.

See [Ocean Token Design: Structure and Behavior](https://github.com/oceanprotocol/research/tree/master/17-permissionless-incentive) for more information about the Ocean token design including the incentive mechanism design.

This OEP defines the architecture, modules and interfaces for network reward distribution  in the settings of IPFS storage. This OEP is on-going work and may be updated frequently along the way.

## Motivation

The main motivations of this OEP are to provide more detailed technical specifications:

* illustrate the high-level architecture to include Ocean components that need changes;
* specify the workflow to generate and distribute network rewards;
* detail the main characteristics of each different modules;
* define the attributes of interfaces between modules;
* identify the new functionalities and attributes that SHOULD be integrated.


# Actors and Technical Components

* SOURCERS - Provide access to assets and/or services
* PROVIDERS - Provide storage in IPFS network and serve the download request
* CONSUMERS - Want to get access to assets and/or services
* MARKETPLACES - Store the DDO (including metadata) associated with the assets and/or services

The following technical components are involved with the publishing flow or the consumption flow:

* MARKETPLACE - Exposes a web interface allowing users to publish and purchase assets. Also facilitates the discovery of assets. An example MARKETPLACE web interface is [Pleuston](https://github.com/oceanprotocol/pleuston).
* [SQUID](https://github.com/oceanprotocol/dev-ocean/blob/master/doc/architecture/squid.md) - Software library encapsulating the Ocean Protocol business logic. It's used to interact with all the components & APIs of the system. It's currently implemented in the following packages:
  * [squid-js](https://github.com/oceanprotocol/squid-js) - JavaScript version of SQUID to be integrated with front-end applications.
  * [squid-py](https://github.com/oceanprotocol/squid-py) - Python version of SQUID to be integrated with back-end applications. The primary users are data scientists.
  * [squid-java](https://github.com/oceanprotocol/squid-java) - Java version of SQUID to be integrated with [JVM](https://en.wikipedia.org/wiki/Java_virtual_machine) applications. The primary users are data engineers.
* [KEEPER CONTRACTS](https://github.com/oceanprotocol/keeper-contracts) - Provide the Service Agreement business logic.
* [SECRET STORE](https://github.com/oceanprotocol/parity-ethereum) - Included as part of the Parity Ethereum client. Allows the PUBLISHER to encrypt the asset URL. Integrates with the SA to authorize (on-chain) the decryption of the asset URL by the CONSUMER.
* [BRIZO](https://github.com/oceanprotocol/brizo) - Microservice to be executed by PUBLISHERS. It exposes an HTTP REST API permitting access to PUBLISHER assets or additional services such as computation.
* [AQUARIUS](https://github.com/oceanprotocol/aquarius) - Microservice to be executed by MARKETPLACES. Facilitates creating, updating, deleting and searching the asset metadata registered by the PUBLISHERS. This metadata is included as part of a DDO (see [OEP-7](../7/README.md) and [OEP-8](../8/README.md)) and also includes the services associated with the asset (consumption, computation, etc.).



