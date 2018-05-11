title Drops and Ocean Tokens


Ocean_Token->Curation_Market: (1) request to purchase drops
Curation_Market->Ocean_Token: (2) freeze Ocean tokens
Curation_Market->Curation_Market: (3) deduct Ocean tokens from balance
Curation_Market->Drops: (4) credit drops to provider

Drops->Curation_Market: (5) request to sell drops
Curation_Market->Drops: (6) deduct drops from balance
Curation_Market->Curation_Market: (7) credit Ocean tokens to provider
Curation_Market->Ocean_Token: (8) activate Ocean tokens
