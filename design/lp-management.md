# LP Management

As briefly introduced in [Broken link](broken-reference "mention"), LP Management (LPM) contracts are used for liquidity providers to safely automate LP risk control and rebalancing between multiple pools of the same class.

The movement of the liquidity token is automated by a trustless backend service, where the token constraints are programmed in the LP Management contract code. Funds can only move between the LPM contract and its allowed Derion pool addresses, with additional restrictions to prevent griefing attacks.

LPM has the following immutable configurations:

* Operator: the address allowed to move the fund around, `0x0` to allow everyone.
* MIN\_RATIO and MAX\_RATIO: the reserve thresholds of each Long and Short side to the pool reserve where the fund movement can occur.

LPM has a storage map for allowed pool addresses to their factories; the liquidity token can only be moved between the LPM contract and its allowed pool addresses in this map.
