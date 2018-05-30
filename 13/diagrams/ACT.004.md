<!-- Open with websequencediagrams.com -->

```text

title Retire an Actor (ACT.004)

Actor->Agent: Retire an Actor (ACT.004)
Agent->Agent: Validate input

Agent-->Actor:HTTP 400 (invalid params)

Agent<->Dec. VM: Get Actor


Agent->Agent: Actor exists & can be retired?

Agent-->Actor:HTTP 404 (not found)
Agent-->Actor:HTTP 401 (forbidden)

note right of Dec. VM: [MarketplaceId] => DISABLED
Agent->Dec. VM: Retire Actor
Dec. VM->Agent: ACK

Agent->Ocean DB: Retire Actor
Ocean DB->Agent: ACK


Agent->Actor: HTTP 202 (Actor)


```
