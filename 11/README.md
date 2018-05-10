```
shortname: 11/MKT
name: Ocean Curation Market
type: Standard
status: Raw
editor: Fang Gong <fang@oceanprotocol.com>
```

<!--ts-->

Table of Contents
=================

   * [Ocean Curation Market](#ocean-curation-market)
      * [Change Process](#change-process)
      * [Language](#language)
      * [Motivation](#motivation)
      * [Architecture](#architecture)
      * [Modules](#modules)
      * [TODO: Events](#events)
      * [Copyright Waiver](#copyright-waiver)
      
<!--te-->

# Ocean Curation Market <a name="ocean-curation-market"></a>


The Ocean Curation Market is a fundamental component of Ocean Protocol to  in the Ocean Network.


This component is based on [Ocean Protocol technical whitepaper](https://github.com/oceanprotocol/whitepaper), [3/ARCH](../3/README.md) and [4/KEEPER](../4/README.md).

## 1. Change Process <a name="change-process"></a>
This document is governed by the [2/COSS](../2/README.md) (COSS).

## 2. Language <a name="language"></a>
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.


## 3. Motivation <a name="motivation"></a>

Ocean network aims to buld a marketplace for relevant AI-related data. In particular, a curation market is constructed to curate high-quality dataset. The key components include:

* **block rewards**: Ocean network generates block rewards to incentivize users who put stakes on high-quality datasets and guarantee the data availability when requested. 
* **bonding curve**: Curation market adjusts the staking cost (i.e., price of drops  for a specific dataset) using bonding curve, therefore, encouraging earlier curations of high-quality dataset due to lower staking cost. 
* **TCR**: Curation market uses Token Curation Registry (TCR) to maintain a list of high-quality dataset and eject bad actors with malicious behavior. 


## 4. Achitecture <a name="architecture"></a>

The architecture of curation market is illustrated in the below, which includes five essential modules. 

![modular architecture](img/curation_market_overview.jpg)

* **Block Rewards**: Ocean network emits Ocean tokens continously according to the pre-defined schedule, which are block rewards for data providers. The distributed amount of block rewards SHOULD be determined by two factors: 
	* *predicted popularity*: number of provider's stake on the specific dataset;
	* *proofed popularity*: number of times made dataset available.
	
* **Ocean Token and Drops**: Ocean network creates derivative tokens of Ocean Token for each dataset curation market, which is called "drops". 
	* Providers purcahse drops with their Ocean tokens to stake on the dataset.
	* Providers can un-stake by selling their drops for Ocean tokens and pocket the profit.
 
* **Bonding Curve**：Bonding curve defines the relationship between price and total supply for drops. 
	* The drops price shoots up when users purchase drops using Ocean token and increase total supploy. 
	* Increased supply indicates more users bet on the popularity of this dataset. 


* **Data Avaliability**: To earn block rewards, providers MUST make dataset available when requested. 
	* It is desired to have multiple providers for the same dataset so that to guarantee data availability. 
	* When dataset is requested, one provider SHOULD be randomly chosen to transfer the dataset. 
	* All providers MUST have the same probability to be chosen.
	* they SHOULD receive equal block rewards if they have the same stakes.  


* **Token Curation Registry (TCR)**: TCR is a powerful mechanism to maintain the high quality of the dataset in the curation market. 
	* Everyone can challenge any dataset or user, which triggers a voting process. 
	* Every participant in the curation market can vote according to his own opinion. 
	* Depends on the voting result, the dataset or user will be either kept in the marketplace or ejected from the system. 


## 5. Modules <a name="modules"></a>

### 5.1 Block Rewards

**(1) Bock Reward Generation**

The 45% of Ocean tokens are used for block rewards and the schedule defines the amount of block rewards to be emitted:

<img src="img/schedule.jpg" width="600" />

With H=10, the figure in the below shows the releasing speed of block rewards over years:

<img src="img/block_reward.jpg" width="500" />

**(2) Block Reward Distribution**

The block reward function defines the amount of block reward that providers receive from a dataset. 
<img src="img/distribution_function.jpg" width="700" />


**(3) Practical Implementation of Block Reward Distribution**

Ocean network SHOULD NOT reward providers at fixed time intervals, which has high complexity and expensive computational cost. Simply imagine the complexity to transfer Ocean block rewards to each dataset curator through on-chain transaction.  

The practical strategy to distribute block rewards is following:

<img src="img/reward_seq.jpg" width="700" />

* every time a provider makes dataset available to a consumer, marketplace SHOULD distribute block rewards;
* marketplace requests block rewards from token contract;
* token contract MUST release block rewards according to schedule and transfers tokens to marketplace;
* marketpalce SHOULD calculate the block reward distribution to current provider with formula;
* it SHOULD increase the balance of provider accordingly without real transfer so that to avoid the cost of transaction;
* marketplace MUST transfer tokens to provider's account upon the received withdraw request.

Here, ``lazy transfer`` strategy is used to significantly reduce the transaction cost:

* the curation market serves as a escrow account and holds the block rewards for providers. 
* However, it maintains the balance for each provider. 
* When the provider requests to withdraw Ocean tokens, curation market initiates the real transfer transaction.
* As such, curation market avoids frequent transfer of tokens to providers. 


**(4) Interface Functions**

The curation market smart contract SHOULD expose the following public methods:

```solidity
	// request token contract to release block rewards
    function requestBlockReward(uint lastRelease) public returns (bool success) { }
    
    // calculate block rewards distribution and credit to provider
    function calcBlockReward(uint _provider, uint _assetId) public returns (uint reward) { }
    
    // transfer block rewards to provider upon his request of withdraw
    funciton withdrawBlockReward(uint _provider) public returns (bool success) { }
```

### 5.2 Ocean Token and Drops

### 5.3 Bonding Curve

### 5.4 Data Availability

### 5.5 Token Curation Registry



## 5. TODO: Events <a name="events"></a>


### Assignee(s)
Primary assignee(s): @gongf05


### Targeted Release

The implementation of the full Keeper functionality it's planned for the [Alpha release](https://github.com/oceanprotocol/ocean/milestone/4)


### Status
unstable


## 6. Copyright Waiver  <a name="copyright-waiver"></a>
To the extent possible under law, the person who associated CC0 with this work has waived all copyright and related or neighboring rights to this work.
