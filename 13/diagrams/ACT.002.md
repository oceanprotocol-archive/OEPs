<!-- Open with websequencediagrams.com -->

```text
title Get an Actor (ACT.002)


Marketplace->Agent: Get an Actor (ACT.002)
note over Marketplace: Could be any Actor

Agent->Agent: Input validation
Agent-->Marketplace:HTTP 400 (invalid input)

Agent->Dec. VM:Get an actor

note left of Dec. VM: Searching if Actor exists

Dec. VM-->Agent:Actor doesn't exists
Agent-->Marketplace:HTTP 404 (not found)

Dec. VM->Agent: Actor information
Dec. VM->Agent: Actor attributes

Agent->Agent: Is OceanDB enabled?

Agent<-->OceanDB: Get Actor metadata
Agent->Marketplace: HTTP 200 (Actor)

```

