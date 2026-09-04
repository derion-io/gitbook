# Protocol Design

Derion is a set of immutable smart contracts for permissionless leveraged-perpetual markets on any index value, with any leverage, reserved in any ERC-20 token.

* Each pool has three share classes, all fungible tokens: LONG and SHORT, each tracking an index value from an external oracle as a true power pay-off, and an LP class that owns whatever reserve the two pay-offs leave.
* Positions carry a compound leverage of $$k$$. Leverage is gradually reduced (deleveraged) as a side's value approaches the pool reserve, so no position can ever be liquidated and the market can never go bankrupt. Solvency is one inequality on the pool's two coefficients, checked at every state change.
* Liquidity is the LP class itself. Anyone provides it by depositing into a pool through the same public path as every trade, and every fee and funding stream accrues to it in place. The [Vault](../liquidity/vault.md) is one holder among any others. A pool is functional with or without it.
* Every trade is verified at both of the oracle's prices, the time-weighted average and the spot, so whichever is adverse to the trade binds. Nothing has to be declared and there is no direction to guess.
* Pools for any pair can be created by anyone with an oracle feed: a Uniswap v3 pair or a Chainlink feed through the built-in fetcher, a tokenized stock through the stock-token fetcher, or any custom fetcher logic.
* The whole market runs on-chain in all conditions, with or without user interaction, with no backend services, keepers, or permissioned roles.
