# Protocol Design

Derion is a set of immutable smart contracts for permissionless leveraged-perpetual markets on any index value, with any leverage, reserved in any ERC-20 token.

* Each pool has exactly two sides — LONG and SHORT — issued as fungible tokens, tracing an index value from an external oracle (on-chain or off-chain).
* Positions carry a compound leverage of K. Leverage is gradually reduced (deleveraged) as a side's value approaches the pool reserve, so no position can ever be liquidated and the market can never go bankrupt.
* Liquidity lives outside the pool: an external [Vault](../vault/README.md) trades through the same public path as everyone else and earns the funding paid by traders. A pool is functional with or without it.
* Pools for any pair can be created by anyone with an oracle feed — Uniswap v3 by default, or any custom fetcher logic.
* The whole market runs on-chain in all conditions, with or without user interaction, with no backend services, keepers, or permissioned roles.
