Open on [websequencediagrams.com](https://www.websequencediagrams.com/)

```text
title Updating Asset Provider (ASE.006)

# ASE.006
participant Provider
participant Agent
participant Orchestrator
participant Dec VM
participant Ocean DB

Provider->Agent: Provider association update req.

Agent->Agent: Input validation
Agent-->Provider: HTTP 400 (Bad params)


Agent->+Orchestrator: Update Provider Associaton

Orchestrator->Dec VM: Update Asset Provider
Dec VM-->Dec VM: Check if asset, user & association exists
Dec VM-->Dec VM: Access Control
Dec VM-->Orchestrator: Error
Orchestrator-->Agent: Error
Agent-->Provider: HTTP 400 (Invalid asset/user)
Agent-->Provider: HTTP 401 (Forbidden)

Dec VM->Orchestrator: ACK
Orchestrator-->Ocean DB: Update Provider Association (*optional)
Ocean DB-->Orchestrator: ACK

Orchestrator->-Agent: ACK

Agent->Provider:  HTTP 202 (Asset)


```
