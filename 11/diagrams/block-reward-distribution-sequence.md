title Block Reward Distribution Sequence

marketplace->token_contract: (1) query block rewards
token_contract->token_contract: (2) calculate num of released token
note right of token_contract: token release schedule formula
token_contract->marketplace: (3) transfer released token

marketplace->marketplace: (4) calculate block reward distribution
note left of marketplace: block reward distribution formula
marketplace->marketplace: (5) increase provider's token balance without transfer

marketplace->provider: (6) transfer tokens upon provider's request