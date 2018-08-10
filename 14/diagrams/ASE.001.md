Open on [websequencediagrams.com](https://www.websequencediagrams.com/)

```text
title Registering an Asset (ASE.001)

# ASE.001
participant Publisher
participant Agent
participant Keeper VM
participant Ocean DB


Publisher->Publisher: Calculate metadata (Hash)

Publisher->Keeper VM: Asset Register request (pricing, hash)
Keeper VM-->Keeper VM: Validation
Keeper VM->Publisher: ACK

Keeper VM<-->Publisher: Event AssetRegistered
Publisher->Agent: Asset Metadata Register request (assetId, metadata, hash)

Agent->Agent: Input validation

Agent-->Publisher: HTTP 400 (Invalid params)


Agent->Agent: Is Ocean DB enabled?
Agent->Ocean DB: Register Metadata


Ocean DB->Agent: ACK

Agent->Publisher:  HTTP 202 (Asset)


```


