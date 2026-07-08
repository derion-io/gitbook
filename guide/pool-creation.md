# Pool Creation

Derion is an open-market protocol: anyone can create a pool for any asset with an oracle feed. A pool is fully defined by its immutable configuration — the same config always produces the same pool address, so identical pools cannot be deployed twice.

Each pool has the following configs:

* **Reserve Token** (`TOKEN_R`): the settlement token of the pool. Everything — collateral, pay-offs, funding — is denominated in it.
* **Price Oracle** (`ORACLE` / `FETCHER`): the index price of the derivatives. By default a Uniswap V3 pair:
  * Pool Address: the Uniswap V3 pool.
  * Window: the number of seconds for the TWAP. A longer window resists price manipulation better, at the cost of more price spread in volatile markets.
  * Quote Token Index (0 or 1): which of the pair's tokens is the quote.
  * A custom `FETCHER` contract can replace the built-in Uniswap logic to adapt any other oracle.
* **Mark Price** (`MARK`): the square-rooted value of the mark price; it should be close to the current market price, and is best shared between pools of the same oracle.
* **Leverage** (`K`): the compound leverage of the two derivative sides.
* **Interest Half-Life** (`INTEREST_HL`): the [funding](../protocol/funding-rate.md) interest, expressed as the half-life of the engine reserve. Zero disables it.
* **Premium Half-Life** (`PREMIUM_HL`): the decay half-life of the trader imbalance, charged to the crowded side. Zero disables it.
* **Opening Rate** (`OPEN_RATE`): the optional [opening fee](../protocol/opening-fee.md).
* **Provider** (`PROVIDER`): the funding payee — normally the shared [Vault](../vault/README.md), which makes the pool eligible for its depth provision.

## Choosing an interest rate

The interest rate replaces three separate costs that a CEX perp trader pays — fixed interest, variable funding, and the implicit cost of liquidation risk — so it should be set with the underlying's volatility in mind. The oracle pool's Uniswap fee tier is a good market-derived proxy for volatility, since spot LPs self-select higher tiers for more volatile pairs:

| Fee tier | Pool type | Examples | Suggested rate | Rationale |
| -------- | --------- | -------- | -------------- | --------- |
| 0.01% (100) | Stable pairs | USDC/USDT | 0.05%/day | Minimal volatility |
| 0.05% (500) | Major pairs | ETH/USDC, BTC/USDC | 0.1%/day | Covers CEX interest + average funding |
| 0.3% (3000) | Mid-tier pairs | LINK/ETH, UNI/ETH | 0.2%/day | Higher volatility, higher convexity cost |
| 1% (10000) | Volatile / exotic | Memecoins, low-caps | 0.3%/day | High volatility premium |

A rate around **0.1%/day is a good default for major pairs** — comparable to the all-in cost of a CEX perp — with higher rates for volatile assets compensating the liquidity side for the no-liquidation protection traders receive. [LP Economics](../vault/lp-economics.md) derives the break-even funding from volatility from first principles; the short version is that a power-2 pool needs total funding near the underlying's variance (σ²) per year for its liquidity to break even.

The suggested rates above are quoted on **notional**, the way CEX funding is. Interest, however, decays the pool's *reserve*, and a position's notional is roughly its leverage times its reserve value — so the reserve rate is the notional rate times the leverage, and the config half-life is `INTEREST_HL = ln(2) / (rate × leverage)`. For a ×2 pool at 0.1%/day, that comes out to a half-life of about 347 days.
