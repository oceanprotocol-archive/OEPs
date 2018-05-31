Open on [websequencediagrams.com](https://www.websequencediagrams.com/)

```text
title Associating Asset and Provider (ASE.005)

# ASE.005
participant Provider
participant Agent
participant Orchestrator
participant Dec VM
participant Ocean DB

Provider->Agent: Provider association request

Agent->Agent: Input validation
Agent-->Provider: HTTP 400 (Bad params)


Agent->+Orchestrator: Provider Association

Orchestrator->Dec VM: Provider Association
Dec VM->Dec VM: Access Control
Dec VM-->Orchestrator: Forbidden
Orchestrator->Agent: Forbidden 
Agent-->Provider: HTTP 401 (Forbidden)

Dec VM->Orchestrator: ACK
Orchestrator-->Ocean DB: Provider Association (*optional)
Ocean DB-->Orchestrator: ACK

Orchestrator->-Agent: ACK

Agent->Provider:  HTTP 202 (Asset)

```