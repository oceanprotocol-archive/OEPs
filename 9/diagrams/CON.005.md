```text

title Get Consumption Information (CON.005)

participant Consumer
participant Agent
participant Dec. VM

Consumer->Agent: Get Consumption Information (CON.005)
note right of Consumer: Only the Consumer can access

Agent->Agent: Validate contractId
Agent-->Consumer:HTTP 400 (bad request)

Agent->Dec. VM:Get consumption info
Dec. VM->Dec. VM: Contract & Asset exists

Dec. VM-->Agent:Contract doesn't exists
Agent-->Consumer:HTTP 404 (not found)

Dec. VM->Dec. VM: Validate user authorization
Dec. VM-->Agent:User un-authorized
Agent-->Consumer:HTTP 401 (forbidden)

Dec. VM->Dec. VM: Validate state & Consumption info

Dec. VM->Agent: Encrypted consumption info

Agent->Consumer: HTTP 200 (ConsumptionInformation)

Consumer->Consumer: Decrypt using public/private keys



```