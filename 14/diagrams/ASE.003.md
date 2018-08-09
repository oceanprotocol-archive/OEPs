Open on [websequencediagrams.com](https://www.websequencediagrams.com/)

```text
title Updating an Asset (ASE.003)

# ASE.003
participant Publisher
participant Agent
participant Keeper VM
participant Ocean DB

Publisher->Publisher: Calculate metadata (Hash)

Publisher->Agent: Asset Update request

Agent->Agent: Input validation

Agent-->Publisher: HTTP 400 (Invalid input)

Agent->Keeper VM: Asset Update
Keeper VM->Keeper VM: Access Control

Keeper VM-->Agent: Forbidden

Agent-->Publisher: HTTP 401 (Forbidden)


Keeper VM->Agent: ACK

Keeper VM<-->Publisher: Event AssetUpdated

Agent->Ocean DB: Update Metadata (including hash)
Ocean DB->Agent: ACK


Agent->Publisher:  HTTP 202 (Asset)



```






