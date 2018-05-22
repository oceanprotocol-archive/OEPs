```text
title Contract Definition (CON.001)

participant Marketplace
participant Agent
participant Dec. VM
participant Ocean DB

# CON.001

Marketplace->Agent: Contract definition (CON.001)
note right of Marketplace: Marketplace facilitate contract creation\nIt could be the Publisher, Provider or Consumer

Agent->Agent: Check input params
Agent-->Marketplace: HTTP 400 (invalid input)

Agent->Dec. VM: Create Contract
Dec. VM->Dec. VM: Actors & Asset validation

Agent-->Marketplace: HTTP 401 (forbidden)


Dec. VM->Agent: ACK

Agent->Agent: Is Ocean DB enabled?
Agent-->Ocean DB: Create Contract
Ocean DB-->Agent: ACK

Agent->Marketplace: HTTP 202 (Contract)




```
