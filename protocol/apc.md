---
description: Self-deleveraging derivatives
---

# Pricing Curve

Uniswap solves the spot DEX problems.\
Derivable solves the perpetual futures DEX problems.

Perpetual and futures exchanges (CEXs and DEXs) have been struggling with position liquidation and counter-party risks, along with the complication of premium and funding rates pricing, as they depend on many independent factors, including price volatility, funding time, and market balance. Derivable introduces a novel asymptotic power curve that deleverages itself as it approaches the reserve limit. This curve allows the creation of an AMM for leveraged perpetual and future derivatives of any index value without the risks of liquidation or under-collateralization.

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Derivative pools using this curve have the following properties:

* 3 sides: LONG, SHORT, and LIQUIDITY (LP) as 3 fungible tokens.
* LONG and SHORT trace an index value from an external oracle (on-chain or off-chain).
* LONG and SHORT are exposed to compound leverage of K; this leverage is gradually decreased (i.e. deleveraged) as the curve approaches the pool reserve.
* The more LIQUIDITY in a pool, the longer both sides stay in the full leverage range.&#x20;
* All sides can never go to zero, i.e. no liquidation risk and no under-collateralization risk.
* Derivative pools for any token pair can be created by anyone as long as they have an Uniswap pool (v2 or v3).
* Efficiently run on Ethereum mainnet, with no backend services nor permissioned roles.

The detailed mathematics and proofs of the curve will be published along with our Whitepaper in the [IEEE](https://www.ieee.org).
