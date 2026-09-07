---
description: Uniswap v3 and Chainlink
---

# Built-in Fetcher

A pool whose `FETCHER` is the zero address uses the fetcher compiled into the pool logic. One packed `ORACLE` word picks the source and carries its parameters:

| Bits | Field | Meaning |
| --- | --- | --- |
| 255 | `QTI` | Orientation flag, Uniswap v3 only. When clear the price is inverted, so either token of the pair can be the numeraire. |
| 224–254 | reserved | |
| 192–223 | `WINDOW` | Uniswap v3: the TWAP averaging window in seconds, which must be non-zero. Chainlink: the staleness bound in seconds. |
| 160–191 | `DECIMALS` | The selector. Zero means a Uniswap v3 pool; non-zero means a Chainlink aggregator and holds that feed's answer decimals. |
| 0–159 | `ADDRESS` | The Uniswap v3 pool, or the Chainlink aggregator. |

Because `WINDOW` carries two different meanings, set it for whichever branch `DECIMALS` selects. A zero window on the Uniswap v3 branch passes seeding, which reads the spot alone, and then fails every fetch.

### Uniswap v3

Any pair on the network is ready for a Derion pool without extra infrastructure. The spot is the pair's current square-root price and the TWAP the square-root price at its mean tick over `WINDOW`, both scaled to Q128. A longer window resists manipulation better, at the cost of more spread in fast markets.

Uniswap carries prices as square roots, so the pool's exponent applies to $$\sqrt{p}$$: a config `K` gives a pay-off power $$k = K/2$$, and `MARK` must be the square root of the mark price.

<figure><img src="../../.gitbook/assets/image (45).png" alt=""><figcaption><p>Uniswap V3 TWAP Oracle</p></figcaption></figure>

### Chainlink

With `DECIMALS` non-zero the same word names a Chainlink aggregator. A feed returns one directed price, so TWAP and spot are equal and the dual-basis gates collapse to a single pass. The fetcher requires a positive answer, a completed round, no stale round, and an update no older than `WINDOW` seconds. The orientation flag is not read: configure the feed in the orientation you need.

Chainlink prices are plain, so here $$k = K$$ and `MARK` is the mark price itself.

### Cross pairs

`CompositeFetcher` prices a pair that has no direct pool by multiplying two Uniswap v3 legs: the main pool and one of a fixed set of WETH-quoted pools (USDC, USDT, or BTC), selected by an index in the word. Its layout carries both legs' orientation flags and windows. Prices are square-root, as for the built-in Uniswap branch. It is a separate contract and is named as a custom `FETCHER`. It ignores `oracleData` but is not payable, so a transition on such a pool must not send native value together with a non-empty `oracleData`.
