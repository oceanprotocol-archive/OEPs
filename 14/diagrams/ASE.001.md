Open on [websequencediagrams.com](https://www.websequencediagrams.com/)

```text
title Registering an Asset (ASE.001)

# ASE.001
participant Publisher
participant Agent
participant Orchestrator
participant Dec VM
participant Ocean DB

Publisher->Agent: Asset Register request

Agent->Agent: Input validation

Agent-->Publisher: HTTP 400 (Invalid params)

Agent->+Orchestrator: Asset Registering

Orchestrator->Dec VM: Register Asset
Dec VM-->Dec VM: Access Control
Dec VM-->Orchestrator: Forbidden
Orchestrator-->Agent: Forbidden
Agent-->Publisher: HTTP 401 (Forbidden)
Dec VM->Orchestrator: ACK

Orchestrator->Orchestrator: Is Ocean DB enabled?
Orchestrator<-->Ocean DB: Register Asset


Orchestrator->-Agent: ACK

Agent->Publisher:  HTTP 202 (Asset)

```


