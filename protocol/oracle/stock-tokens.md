---
description: Tokenized equities on Robinhood Chain
---

# Stock Tokens

`RobinhoodFetcher` prices a Robinhood Chain stock token, a tokenized US equity, from two independent sources. One immutable instance is deployed per ticker.

### The two legs

**TWAP basis: the Pyth xStock feed.** The corresponding xStock token (the same equity wrapped by a different issuer) has a Pyth price published around the clock from its own markets. That price is normalized onto the Robinhood token:

$$
\text{twap} = \frac{P_{\text{xStock/USD}}}{RR_{\text{xStock/underlying}}} \cdot \frac{1}{P_{\text{quote/USD}}} \cdot \frac{m}{10^{18}}
$$

The redemption-rate feed $$RR$$ strips the xStock wrapper's own drift (its dividend and rebase policy); the optional quote feed reprices dollars into the units of the local pool's quote token; the ERC-8056 multiplier $$m$$ re-applies the Robinhood wrapper's corporate actions. Each of the three is optional in the configuration.

**Spot: the local Uniswap v4 pool.** The stock token's canonical Uniswap v4 pool on Robinhood Chain, read from the PoolManager and rescaled by the decimals offset between the stock token and its quote.

Both legs are returned as **plain** Q128 prices (quote per stock token, in human units), like the Chainlink branch of the [built-in fetcher](builtin.md) and unlike its Uniswap v3 branch. A pool on this fetcher sets `MARK` as a plain price and `K` as the pay-off power itself, $$k = K$$.

### Why two sources

The engine gates every trade at both bases, so manipulating either source alone only worsens the manipulator's own execution. A trader-favorable mispricing needs the local v4 pool and the global xStock market to be wrong in the same direction at the same time. On top of that, the fetcher refuses to price when the two legs disagree by more than `maxDivergeBps` of the lower one; trading halts until they reconverge.

### Corporate actions

A split moves the redemption rate and/or the multiplier, never the composed price. Take a 2:1 split where the xStock rebases (its price halves, $$RR$$ stays 1) and the Robinhood token doubles its multiplier: the TWAP leg is unchanged, the v4 pool's price of the token is unchanged, and quotes through the pool are continuous. The only exposure is transient: the two can update out of sync, and the divergence cap together with the engine's adverse-bound gating contains that window.

### Freshness

Pyth is a pull oracle. Prices reach the chain when someone posts a signed update, and the fetcher accepts two freshness regimes.

**Stored price.** With empty `oracleData` the fetcher reads the receiver's stored price and requires it no older than `maxAge` seconds (`maxAgeRR` for the redemption rate, which updates rarely). Pokes, quotes, and the Vault's NAV marking take this path. A keeper owns it.

**Caller-carried price.** A transactor puts Pyth update blobs in `Param.oracleData`. The fetcher hands them to the receiver's stateless parse, which verifies the signatures and returns the price without storing it, and requires the publish time to fall inside $$[\text{now} - \text{maxAgeTrade},\ \text{now} + 15\,\text{s}]$$. Nothing is written, so the keeper still owns the stored price the empty-data paths read. Signature verification plus the window mean a caller can neither forge a price nor cherry-pick a stale one.

`maxAgeTrade` is a per-pool field carried in the pool's `ORACLE` word, so pools sharing one fetcher instance can each tune their own window:

| Bits | Field |
| --- | --- |
| 192–255 | reserved |
| 160–191 | `maxAgeTrade` in seconds; 0 falls back to `maxAge` |
| 0–159 | the stock token address, checked against the instance |

The 15-second publish-ahead tolerance covers clock skew between the sequencer and Pythnet. It is a property of the chain, not of a market, so it is a constant rather than a per-pool field.

### Update fees

Pyth receivers may charge for updates. The pool forwards the caller's native value to the fetcher only on the fresh-update path (non-empty `oracleData`); the fetcher pays exactly the receiver's quoted fee and refunds the remainder, which the pool recovers for the reserve leg or returns to the caller. The same path works against a fee-less and a fee-charging receiver. Underpaying reverts.

### Validation

Every Pyth value, the price, the redemption rate and the quote feed alike, must be positive with an exponent in $$[-59, 0]$$ and, when the cap is set, a confidence interval within `maxConfBps` of its own value. The v4 read fails closed: an uninitialized pool, or an overflow at an absurd raw price ratio, reverts rather than returning a wrong number. The `ORACLE` word's stock token must match the instance's.

### Initialization

The seeding price for `init` is the v4 spot alone, with no Pyth involvement. A pool can therefore be created before its feed has ever been stored on the receiver, and a trade-only ticker, one the keeper never pushes, still trades through caller-carried data.

### Configuration

The per-ticker immutables:

| Field | Meaning |
| --- | --- |
| `poolManager`, `poolId`, `tokenIs0` | The Uniswap v4 PoolManager, the canonical stock-token/quote pool, and whether the stock token is `currency0` of it. |
| `decimalsOffset` | Stock token decimals minus quote decimals (18 − 6 = 12 for a Robinhood token over USDG). |
| `pyth`, `priceId` | The Pyth receiver and the `Crypto.<TICKER>X/USD` feed. |
| `rrId` | The `Crypto.<TICKER>X/<TICKER>.RR` redemption-rate feed; 0 to skip. |
| `quoteId` | The `Crypto.<QUOTE>/USD` feed to price the TWAP leg in the pool's quote units; 0 assumes the quote holds $1. |
| `multiplier` | The ERC-8056 `uiMultiplier()` source (the stock token itself); 0 to skip. |
| `stockToken` | Sanity-checked against the `ORACLE` word. |
| `maxAge`, `maxAgeRR` | Stored-price staleness bounds in seconds. |
| `maxConfBps` | Confidence cap in basis points of the price; 0 disables. |
| `maxDivergeBps` | TWAP/spot divergence cap in basis points of the lower; 0 disables. |

### Deployment prerequisites

To be verified on-chain before any mainnet pool:

* A Pyth receiver on Robinhood Chain (chain id 4663) that speaks the standard interface and verifies update payloads. The receiver at `0xa80258Eea4BA0865610eb239045737D08929c40b` did so when last inspected, in August 2026: it takes permissionless fee-less updates and carries NVDAX, SPYX, GOOGLX, SPCXX, USDG, and ETH feeds. Its implementation is unverified and sits behind an owner-upgradable proxy, so the oracle operator is trusted. Inspect it again before deploying against it.
* A keeper, or a router bundling `updatePriceFeeds` ahead of `transition`, keeping the stored feeds within `maxAge` and `maxAgeRR`. Hermes, the Pyth price service the updates are pulled from, needs a paid plan for production use.
* The canonical, deepest v4 pool for the ticker, whose hook must not distort its slot0 semantics.
* The ERC-8056 `uiMultiplier()` name and 1e18 scale, per Robinhood's stock-token documentation.

### What the fetcher does not do

The underlying stock trades six and a half hours a day; the xStock feed and the local v4 pool trade around the clock, and off-hours prices are thin. The divergence cap and the dual-basis gate contain manipulation of one source. They do not stop a trader who is right about where the stock will open, and there is no market-hours switch: a stale Pyth price halts trading, which is the closest thing to a pause the pool has. Scheduled moves (earnings, macro prints) are a problem for the pool's parameters rather than for the oracle; see [Pool Parameters](../../guide/pool-parameters.md).
