# Mint Module

The `jklmint` module is responsible for the management of the native JKL token within the Jackal Protocol. This module handles the issuance, distribution, and inflation of JKL tokens, ensuring a stable and secure token economy.

### Overview

The `jklmint` module manages the following aspects of the JKL token:

1. **Token Issuance**: Determines the initial token supply and distribution.
2. **Inflation**: Manages the annual inflation rate, maintaining a consistent token supply growth.
3. **Rewards Distribution**: Handles the allocation of newly minted tokens as rewards for validators and delegators.

#### Parameters

The `jklmint` module uses the following parameters to manage the JKL token economy:

* `mint_denom`: the token to print
* `mint_decrease`: the amount in % the production of tokens will slow over time
* `tokens_per_block`: the base value of tokens to print per block
* `dev_grants_ratio`: the ratio in % of how much of the inflation should go to the developer grants
* `staker_ratio`: the ratio in % of how much of the inflation should go to the stakers
* `storage_stipend_address`: which address should the storage stipend go to
* `storage_provider_ratio`: the ratio in % of how much inflation should go to the storage provider stipend
