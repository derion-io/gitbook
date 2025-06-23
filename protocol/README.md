# Protocol Design

Derion Protocol is a set of smart contracts designed for the creation of leverage perpetual pools for any index value, with any leverage, and reserved with any ERC-20 token.

* 3 sides: LONG, SHORT, and LIQUIDITY (LP) as 3 fungible tokens.
* LONG and SHORT trace an index value from an external oracle (on-chain or off-chain).
* LONG and SHORT are exposed to compound leverage of K; this leverage is gradually decreased (i.e., deleveraged) as the curve approaches the pool reserve.
* The market (or pool) is functional in all market conditions, with or without user interaction. Positions and market liquidity are designed to asymptotically approach zero but never reach it, thereby eliminating the risk of liquidation.
* Derivative pools for any token pair can be created by anyone as long as they have an oracle feed, like Uniswap v2, v3 or Chainlink.
* Efficiently run on Ethereum mainnet, with no backend services nor permissioned roles.
