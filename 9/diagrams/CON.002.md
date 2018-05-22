```text
title Retrieve Contract Information (CON.002)


participant Any
participant Agent
participant Dec. VM
participant Ocean DB

Any->Agent: Retrieve Contract (CON.002)
note right of Any: Could be sent by any Actor

Agent->Agent: Validate contractId
Agent-->Any:HTTP 400 (bad request)

Agent->Dec. VM:Get contract info
note left of Dec. VM: Searching if Contract exists

Dec. VM-->Agent:Contract doesn't exists
Agent-->Any:HTTP 404 (not found)

Dec. VM->Agent: Contract information

note over Ocean DB: * optional
Ocean DB<-->Agent: Contract information
Agent-->Agent: Build model

Agent->Any: HTTP 200 (Contract)
```