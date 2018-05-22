```text
title Authorize consumption (CON.004)

participant Provider
participant Agent
participant Dec. VM
participant Ocean DB

# CON.003


Provider->Agent: Contract definition (CON.004)
note right of Provider: Only the provider can authorize

Agent->Agent: Check input params
Agent-->Provider: HTTP 400 (invalid input)

Agent->Dec. VM: Authorize
Dec. VM->Dec. VM: Are you the Provider?
Dec. VM->Dec. VM: Contract & State validation
Dec. VM-->Agent: Error

Agent-->Provider: HTTP 401 (forbidden)

Dec. VM->Dec. VM: Update State & Consumption info
Dec. VM->Agent: ACK

Agent->Agent: Is Ocean DB enabled?
Agent-->Ocean DB: Update state
Ocean DB-->Agent: ACK


Agent->Provider: HTTP 202 (Contract)
```