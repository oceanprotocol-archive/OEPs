Open on [websequencediagrams.com](https://www.websequencediagrams.com/)

```text
title Updating an Asset (ASE.003)

# ASE.003
participant Publisher
participant Agent
participant Orchestrator
participant Dec VM
participant Ocean DB

Publisher->Agent: Asset Update request

Agent->Agent: Input validation

Agent-->Publisher: HTTP 400 (Invalid input)

Agent->+Orchestrator: Asset Update

Orchestrator->Dec VM: Update Asset

Dec VM->Dec VM: Access Control
Dec VM-->Orchestrator: Forbidden
Orchestrator-->Agent: Forbidden
Agent-->Publisher: HTTP 401 (Forbidden)


Dec VM->Orchestrator: ACK

Orchestrator-->Ocean DB: Update Asset (* optional)
Ocean DB-->Orchestrator: ACK

Orchestrator->-Agent: ACK

Agent->Publisher:  HTTP 202 (Asset)






```






