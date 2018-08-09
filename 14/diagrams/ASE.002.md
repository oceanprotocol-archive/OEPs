Open on [websequencediagrams.com](https://www.websequencediagrams.com/)

```text
title Retrieve Asset Metadata (ASE.002)

participant Any
participant Agent
participant Ocean DB

Any->Agent: Get Metadata (ASE.002)
note right of Any: Could be sent by any Actor

Agent->Agent: Validate assetId
Agent-->Any:HTTP 400 (bad request)

Agent-->Agent: Is Ocean DB enabled?
Agent->Ocean DB:Get asset Metadata (assetId)

Ocean DB-->Agent:Asset doesn't exists
Agent-->Any:HTTP 404 (not found)

Ocean DB->Agent: Asset information


Agent-->Agent: Build Asset model

Agent->Any: HTTP 200 (Asset)



```