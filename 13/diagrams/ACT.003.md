<!-- Open with websequencediagrams.com -->

```text

title Update an Actor (ACT.003)

Actor->Agent: Update an Actor (ACT.003)
note over Actor: Marketplace could do as proxy
Agent->Agent: Validate input
Agent->Agent: Is Ocean DB enabled?

Agent-->Actor:HTTP 400 (invalid params)

Agent<->Dec. VM: Get Actor

Agent->Agent: Actor exists & can update?

Agent->Dec. VM: Update attributes (*optional)

Agent-->Actor:HTTP 404 (not found)
Agent-->Actor:HTTP 401 (forbidden)


Agent->Ocean DB: Update Actor Metadata
Ocean DB->Agent: Creation ACK


Agent->Actor: HTTP 202 (Actor)

```



