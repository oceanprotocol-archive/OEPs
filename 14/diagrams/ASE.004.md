Open on [websequencediagrams.com](https://www.websequencediagrams.com/)

```text
title Retiring an Asset (ASE.004)


# ASE.004
participant Publisher
participant Agent
participant Keeper VM
participant Ocean DB

Publisher->Keeper VM: Retire Asset (assetId)

Keeper VM->Keeper VM: Input validation
Keeper VM-->Publisher: Error (Invalid params)

Keeper VM->Keeper VM: Access Control
Keeper VM-->Publisher: Error (Forbidden)

Keeper VM->Publisher: ACK



Keeper VM<-->Publisher: Event AssetRetired
Keeper VM<-->Agent: Event AssetRetired

Agent->Ocean DB: Retire Asset
Ocean DB->Agent: ACK


```