# Protocol

Derivable Protocol is a set of smart contracts designed for the creation of leverage perpetual pools for any index value, with any leverage, and reserved with any ERC-20 token.

Derivable pools have the following properties:

* 3 sides: LONG, SHORT, and LIQUIDITY (LP) as 3 fungible tokens.
* LONG and SHORT trace an index value from an external oracle (on-chain or off-chain).
* LONG and SHORT are exposed to compound leverage of K; this leverage is gradually decreased (i.e., deleveraged) as the curve approaches the pool reserve.
* The more LIQUIDITY in a pool, the longer both sides stay in the full leverage range.&#x20;
* All sides can never go to zero, i.e., no liquidation and infinite liquidity.
* Derivative pools for any token pair can be created by anyone as long as they have an Uniswap pool (v2 or v3).
* Efficiently run on Ethereum mainnet, with no backend services nor permissioned roles.
