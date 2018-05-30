<!-- Open with websequencediagrams.com -->

```text

title Register new Actor (ACT.001)

participant Actor
participant Agent
participant Accounts Manager
participant Dec. VM
participant Ocean DB

Actor->Agent: Registering an Actor (ACT.001)
Agent->Agent: Input Validation
Agent-->Actor:HTTP 400 (bad request)
Agent->Accounts Manager:Create Account and Wallet
Accounts Manager->Agent:Account information
Agent<->Dec. VM: Check if actor already exists
Agent->Agent: Already exists?

Agent-->Actor:HTTP 422 (already exists)

Agent->Dec. VM: Create Account

Dec. VM->Agent: Register ACK

Agent->Dec. VM: Register attributes (*optional)

Agent->Agent: Is Ocean DB enabled?
Agent<-->Ocean DB: Create Account

Agent->Actor: HTTP 202 (Actor)

```