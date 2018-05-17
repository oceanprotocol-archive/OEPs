```
shortname: 11/MKT
name: Ocean Curation Market
type: Standard
status: Raw
editor: Fang Gong <fang@oceanprotocol.com>
contributors: Aitor Argomaniz <aitor@oceanprotocol.com>
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
      * [Reference](#reference)
      * [Copyright Waiver](#copyright-waiver)
      

# Ocean Curation <span style="color:red">Proofs</span> Market <a name="ocean-curation-market"></a>


The Ocean Curation Market is a critical cornerstone of Ocean Protocol. This document bridges the theory in technical whitepaper[1] and the smart contract implementation in practice. 

The proposed solutions are based on Ocean Protocol technical whitepaper [1], Trent's talk [2] & blog [3], [3/ARCH](../3/README.md) and [4/KEEPER](../4/README.md).

## 1. Change Process <a name="change-process"></a>
This document is governed by the [2/COSS](../2/README.md) (COSS).

## 2. Language <a name="language"></a>
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] \[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, as shown here.


## 3. Motivation <a name="motivation"></a>

Ocean network aims to build a marketplace for relevant AI-related data. Curation market coordinates participants to maintain high-quality assets (mainly datasets) and the normal operations of marketplace, which is a fundamental component of Ocean Network. 

<!--
The key components include:

* **Block Rewards**: Ocean network generates block rewards to incentivize users who bet on dataset with stakes and ensure the data availability when requested. 
* **Bonding Curve**: Curation market dynamically adjusts the stake cost (i.e., drops price of dataset) using bonding curve, therefore, encouraging earlier curations of dataset with lower staking cost. 
* **TCR**: Curation market uses Token Curation Registry (TCR) to maintain high-quality dataset and eject bad actors with malicious behavior. 
-->

## 4. Architecture <a name="architecture"></a>

The architecture of curation market is illustrated in the below, which includes five essential modules. 

![modular architecture](img/curation_market_overview.jpg)

* **Block Rewards**: Ocean network emits Ocean tokens continuously to incentivize providers according to the pre-defined schedule, which are called "block rewards". The distributed amount of block rewards SHOULD be determined by two factors: 
	* *Predicted popularity*: number of provider's stake on the dataset;
	* *Proofed popularity*: number of times made dataset available.
	
* **Ocean Token and Drops**: Ocean network creates derivative tokens of Ocean Token for each curation market, which is called "**drops**". 
	* Providers purchase drops with their Ocean tokens to stake on the asset.
	* Providers can un-stake by selling their drops for Ocean tokens and realize the profit.
 
* **Bonding Curve**: Bonding curve defines the relationship between price and total supply of drops. 
	* The drops price shoots up when more users purchase drops using Ocean token and increase the total supply. 
	* Increased supply indicates more users bet on the popularity of asset. 


* **Data Availability**: To earn block rewards, providers MUST make assets available when requested. 
	* It is desired to have multiple providers for the same asset so that to guarantee data availability. 
	* When an asset is requested, one provider SHOULD be randomly chosen to transfer the asset. 
	* All providers MUST have the same probability to be chosen.
	* They SHOULD receive equal block rewards if they have the same stakes.  


* **Token Curation Registry (TCR)**: TCR is a powerful mechanism to maintain the high quality of the asset in the curation market. 
	* Everyone can challenge any asset or user, which triggers a voting process. 
	* Participants in the curation market can vote according to their own opinions. 
	* Depends on the voting result, the asset or actor will be either kept in the marketplace or ejected from the system. 


## 5. Modules <a name="modules"></a>

### 5.1 Block Rewards

**(1) Bock Reward Generation**

The 45% of total Ocean tokens are used for block rewards and the schedule determines the amount of block rewards available to be emitted at any timepoint [1]:

<img src="img/schedule.jpg" width="600" />

With **H=10**, the figure below shows the speed of releasing block rewards over years:

<img src="img/block_reward.jpg" width="500" />

**(2) Block Reward Distribution**

The block reward function determines the amount of block reward that providers expect to receive[1]: 

<img src="img/distribution_function.jpg" width="700" />


**(3) Practical Implementation of Block Reward Distribution**

Ocean network SHOULD NOT reward providers at fixed time intervals, which has high complexity and expensive computational cost. Simply imagine the complexity to transfer Ocean block rewards to each provider through token-transfer transactions.  

The practical strategy to distribute block rewards is [following](diagrams/block-reward-distribution-sequence.md):

<img src="img/reward_seq.jpg" width="700" />


* Every time a provider delivers dataset to a consumer, marketplace SHOULD distribute block rewards;
* Marketplace requests block rewards from token contract;
* Token contract MUST release block rewards according to schedule and transfers tokens to marketplace;
* Marketplace SHOULD calculate the block reward distribution to current provider with formula;
* It SHOULD increase the balance of provider accordingly without token transfer which avoids the cost of token-transfer transactions;
* Marketplace only transfer tokens to provider's account upon the withdraw-token request.

Noted that **"lazy transfer"** strategy is used to significantly reduce the transaction cost:

* The curation market serves as a escrow account and holds the block rewards for providers. 
* It maintains the correct balance record for each provider. 
* When the provider requests to withdraw Ocean tokens, curation market transfer tokens to provider which SHOULD match the balance amount.
* Curation market avoids frequent token transfer to providers and saves transaction cost. 


**(4) Smart Contract Interface Functions**

The curation market smart contract SHOULD include data structure to keep the records of providers and balances for each asset:

```solidity
// Asset struct has an array of providers Id
struct Asset {
	...
	uint[] providerId;
	...
}

// Provider struct has balance of Ocean Tokens
struct Provider {
	...
	uint balance;
	...

}

// Hashtable to map provider Id to provider struct
mapping(uint => Provider) id2provider

```

The contract SHOULD expose the following public methods:

```solidity
	// request token contract to release block rewards
    function requestBlockReward() public returns (bool success) { }
    
    // calculate block rewards distribution and credit to provider
    function calcBlockReward(uint _provider, uint _assetId) public returns (uint reward) { }
    
    // transfer block rewards to provider upon his request of withdraw
    function withdrawBlockReward(uint _provider) public returns (bool success) { }
```

### 5.2 Ocean Token and Drops

Each asset creates its own curation market in Ocean network and needs its native tokens, which is called "drops". In fact, drops are *derivative tokens* of Ocean Tokens which means drops can be exchanged from/to Ocean Tokens. 

As shown in the figure: 

* Ocean tokens can be used to purchase drops for different asset (i.e., Drops 1, ..., Drops K) which represent user's stakes on asset / service. 
* These drops can be sold for Ocean tokens so that users can un-stake and realize their profits.

<img src="img/drops.jpg" width="600" />


Curation market smart contract SHOULD record the drops balance for each provider. 

```solidity
// Asset struct can map provider id to his drops balance
struct Asset {
	...
	uint[] providerId;
	...
	mapping(uint => uint) id2drops;
	...
}
```

To implement the exchange between Ocean tokens and Drops, we adopt the **2-Way peg approach** similar to the mechanism behind [RootStock](https://faq.rsk.co/hrf_faq/what-is-the-2-way-peg/):

* There is no single transaction to transfer Ocean tokens or Drops;
* To purchase Drops, some Ocean tokens are locked and the corresponding Drops with the same value are unlocked.
* To sell Drops, the Drops get locked again and the Ocean tokens with the same value are unlocked.
* Both procedures are handled by smart contract to ensure the security.

In the practical implementation, curation market smart contract MUST execute following operations:

* Lock the Ocean tokens that are used to purchases drops;
* Reduce user's balance of Ocean token by the same amount;
* Increase user's balance of drops by the amount that has an equal value with the locked Ocean tokens; 
* Reverse above operations when users un-stake.

The process can be illustrated with [below figure](diagrams/drops-and-ocean-tokens.md):

<img src="img/token_drops.jpg" width="700" />


The smart contract SHOULD have interface functions:

```solidity
// purchase drops using Ocean token
function purchaseDrops(uint _assetId, uint _providerId, uint _amount) public returns (bool success) { }

// sell drops for Ocean token
function sellDrops(uint _assetId, uint _providerId, uint _amount) public returns (bool success) { }
```    

To lock and unlock Ocean tokens, it can be implemented with `Allowance` variable, which represents the available amount of Ocean tokens to be transferred by curation market. 

* **Purchase drops**: lock Ocean tokens by reducing the `allowance` value;
* **Sell drops**: unlock Ocean tokens by increasing `allowance` with the same value as locked Drops.


### 5.3 Bonding Curve

The rate between Ocean tokens and Drops is determined by Bonding Curve as shown in the below:
<img src="img/bonding_curve.jpg" width="600" />

Clearly, the price of drops depends on the total supply:

* More users buy drops and total supply increases => drops price shoots up;
* More users sell drops and total supply decreases => drops price plummets;

Curation market smart contract SHOULD have bonding curve function as:

```solidity
struct Asset {
	...
	uint drops_supply;
	...
}

// query the price of drops at current supply
function bondingCurve(uint _supply) public returns (uint price) { }
```


### 5.4 Data Availability

There are multiple providers for the same asset, which provide better data availability. When data is requested, one provider SHOULD be randomly chosen with **uniform-distributed sampling** to provide the dataset.

Since all providers have the same probability to be chosen, they SHOULD receive equal block rewards with the same amount of stakes on dataset.

<img src="img/data_supply.jpg" width="600" />

Note that *random number generator is not available* in Ethereum Virtual Machine, as the code will be ran on multiple nodes, on different time. It does not make sense to generate different random numbers on different nodes at different time in Ethereum network. 

Instead, the AGENT client SHOULD randomly choose a provider to request the dataset. The curation market smart contract only need to log the provider, who transfers the dataset to consumer, and increment the counter of dataset download times.

```solidity
// Asset has provider list and their corresponding download times 
struct Asset {
	...
	uint[]	providerId;
	unit	totalDownload;
	unit	lastProvider;
	...
}

// hashtable mapping assetId to asset struct
mapping(uint => Asset) id2asset;

// return the list of all providers
function queryProviders(uint _assetId) public returns (uint[] _list) { }

// log the provider for current download and increment the counter of #download
function updateDownload(uint _assetId, uint _providerId) public returns (bool success) { }
```

* The function `queryProviders ` SHOULD return the list of providers to AGENT client so that AGENT can randomly choose one provider (i.e., unifrom-distributed sampling) to request the dataset. 
* The function `updateDownload` SHOULD increment `totalDownload` of the dataset and log the provider Id into `lastProvider`. 

### 5.5 Token Curation Registry

Curation market coordinates the community to maintain the high-quality data together and Token Curation Registry (TCR) is used for that purpose in Ocean network:

<img src="img/TCR.jpg" width="600" />

The [workflow of TCR](diagrams/tcr-workflow.md) can be illustrated as below:
 
<img src="img/TCR_flow.jpg" width="700" />


* Any user can apply for adding new asset or challenge existing assets or actors;
* All participants can vote to support or against the asset or actor;
* Curation market reveals the result after the voting period is closed;
* The majority party win the voting and rewards;
* Curation market keep or eject the asset or actor according to the voting result.


Smart contract SHOULD have data struct and interface functions to implement TCR in Ocean network:

```solidity
// Challenge data struct
struct Challenge {
	uint		id;
	address		owner;
	uint		stake;
	bool		resolved;
	mapping(address => bool) claimedToken; //whether a voter has claimed token reward
}

// hashtable mapping the changeId to associated Challenge data
mapping(uint => Challenge) id2challenges;

// apply for new dataset with deposit
function apply(uint _amount, bytes32 _dataHash) returns (uint challengeId) { }

// start a poll for challenging dataset / actor and send deposit
function challenge(uint _amount, bytes32 _dataHash) returns (uint challengeId) { }

// vote for or against 
function voteChallenge(uint _challengeId, uint _opinion) returns (bool success) { }

// reveal voting result and resolve the challenge
function resolveChallenge(uint _challengeId) returns (bool success) { }

// determine rewards to the winning party in a challenge
function determineReward(uint _challengeId) returns (uint tokens) { } 
```




## 6. TODO: Events <a name="events"></a>


### Assignee(s)
Primary assignee(s): @gongf05


### Targeted Release

The implementation of the full Keeper functionality it's planned for the [Alpha release](https://github.com/oceanprotocol/ocean/milestone/4)


### Status
unstable

## 7. Reference <a name="reference"></a>

* [1][Ocean Protocol Technical Whitepaper](https://oceanprotocol.com/tech-whitepaper.pdf)
* [2][Trent McConaghy, Co-Founder - Curated Proof Markets & Token-Curated Identities](https://www.youtube.com/watch?v=LxkvJmh7t0Y)
* [3][Curated Proofs Markets: A Walk-Through of Ocean’s Core Token Mechanics](https://blog.oceanprotocol.com/curated-proofs-markets-a-walk-through-of-oceans-core-token-mechanics-3d50851a8005)


## 8. Copyright Waiver  <a name="copyright-waiver"></a>
To the extent possible under law, the person who associated CC0 with this work has waived all copyright and related or neighboring rights to this work.
