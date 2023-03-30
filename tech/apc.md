---
description: Self-leverageing derivatives
---

# Asymptotic Power Curve

Perpetual and Futures exchanges (both CEX and DEX) have been struggling with the calculation of premium and funding rates as they depend on many different factors, including market price, time, and market balance. Derivable introduces a novel power curve that deleverages itself as it approaches the reserve limit. This curve allows the creation of AMM for leveraged perpetual and future derivatives of any index value without the risks of liquidation or under-collateralization.

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Derivative pools using this curve have the following properties:

* 3 sides: LONG, SHORT, and LIQUIDITY (LP) as 3 fungible tokens.
* LONG and SHORT trace an index value from an external oracle (on-chain or off-chain)
* LONG and SHORT are exposed to a configurable leverage K, this leverage is gradually decreased (deleveraged) as the curve is approaching the pool reserve.
* The more LIQUIDITY in a pool, the longer both sides stay in the full leverage range.&#x20;
* All sides can never go to zero, i.e. no liquidation risks and no under-collateralization risks.

The final formula for the curve will be published along with our Whitepaper in [IEEE](https://www.ieee.org).
