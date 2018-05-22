```text
title Sign Contract (CON.003)

participant Any
participant Agent
participant Dec. VM
participant Ocean DB

# CON.003


Any->Agent: Sign Contract (CON.003)
note right of Any: Any party related\n with the contract

Agent->Agent: Check input params
Agent-->Any: HTTP 400 (invalid input)

Agent->Dec. VM: Sign Contract
Dec. VM->Dec. VM: Contract & Signature validation

Agent-->Any: HTTP 401 (forbidden)

Dec. VM->Agent: ACK

Agent->Agent: Is Ocean DB enabled?
Agent-->Ocean DB: Sign Contract
Ocean DB-->Agent: ACK


Agent->Any: HTTP 202 (Contract)

```