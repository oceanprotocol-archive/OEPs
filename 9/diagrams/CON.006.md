```text

title Revoke Signature (CON.006)

participant Any
participant Agent
participant Dec. VM
participant Ocean DB

# CON.003


Any->Agent: Revoke Signature (CON.006)
note right of Any: Any party related\n with the contract

Agent->Agent: Check input params
Agent-->Any: HTTP 400 (invalid input)

Agent->Dec. VM: Revoke Signature
Dec. VM->Dec. VM: Contract, State & Signature validation
Dec. VM-->Agent: Forbidden
Agent-->Any: HTTP 401 (forbidden)


Dec. VM->Agent: ACK

Agent->Agent: Is Ocean DB enabled?
Agent-->Ocean DB: Update state
Ocean DB-->Agent: ACK


Agent->Any: HTTP 202 (Contract)


```