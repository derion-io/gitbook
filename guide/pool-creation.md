# Pool Creation

Derion is an open-market protocol: anyone can create a pool for any asset with an oracle feed. A pool is fully defined by its immutable configuration, and the same config always produces the same pool address, so identical pools cannot be deployed twice. Deployer and initiator earn nothing for it and hold no privilege afterward.

## Configuration

* **Reserve Token** (`TOKEN_R`): the settlement token of the pool. Everything, collateral, pay-offs, funding, is denominated in it.
* **Price Oracle** (`FETCHER`, `ORACLE`): the index price of the derivatives.
  * Zero `FETCHER` selects the [built-in fetcher](../protocol/oracle/builtin.md). The `ORACLE` word then names a Uniswap v3 pair (with its quote-token orientation and TWAP window) or a Chainlink aggregator (with its decimals and staleness bound). A longer TWAP window resists manipulation better at the cost of more spread in volatile markets.
  * A [stock-token](../protocol/oracle/stock-tokens.md) pool names the per-ticker `RobinhoodFetcher` instance and puts the stock token address in the low bits of `ORACLE`, optionally with a per-pool freshness window above it.
  * Any other [custom fetcher](../protocol/oracle/custom.md) can adapt any price source.
* **Leverage** (`K`) and **Mark Price** (`MARK`): the pay-off power and the price the power is centered on. Both are read in the fetcher's price convention. With a square-root-price fetcher (Uniswap v3) `K` is twice the pay-off power and `MARK` is the square root of the mark price; with a plain-price fetcher (Chainlink, stock tokens) `K` is the power itself and `MARK` the price. `MARK` should be close to the current market price and is best shared between pools on the same oracle. Practical range for `K` is 1 to 32; the contract does not bound it, but far from the mark a very large `K` falls back to a slow power loop and can exhaust gas, and high-power pools are the ones sensitive to the saturated-and-diverged liveness corner.
* **Interest Half-Life** (`INTEREST_HL`): [interest](../protocol/funding-rate.md) as the half-life of each side's reserve, in seconds. Zero disables it.
* **Premium Half-Life** (`PREMIUM_HL`): the decay half-life of the side imbalance, charged to the crowded side, in seconds. Zero disables it.
* **Opening Rate** (`OPEN_RATE`): the optional [opening fee](../protocol/opening-fee.md), as the fraction of the gross payment that becomes position, x128. `2^128` is no fee.

There is no provider field. Liquidity is the pool's own [LP class](../liquidity/lp-class.md), which anyone, the Vault included, holds by depositing.

## Seeding

A deployed pool cannot be used until `init` seeds it with a state $$\langle R, \alpha, \beta \rangle$$ and the reserve $$R$$ paid in. At the fetcher's spot price the two curves split $$R$$ into $$r_A$$, $$r_B$$, and the residual $$r_C$$, and all three must clear the minimum reserve (10⁶ wei each). A residual below zero means the seed is insolvent and the call reverts. A simple choice is a third each: pick $$\alpha$$ so that $$r_A = R/3$$ and $$\beta$$ so that $$r_B = R/3$$ at the seeding price, using the inverse of the curve.

All three seeds are minted to an address nobody controls. They protect the pool against share-inflation attacks and are the permanent supplies the per-share gates divide by, and they are never returned. Keep the seed small.

The Factory's `deploy` runs deployment and `init` in one transaction, and `deployWithStrategy` additionally writes a Vault strategy that extends an existing one with the new pool, ready to be voted for ([Strategy Governance](../liquidity/governance.md)). See the [Pool API](../contracts/api.md).

## Choosing the parameters

Leverage, the two half-lives, the fee, and the Vault's headroom and ramp for the pool are one decision, driven by the asset's volatility and event calendar. The LP class is short a straddle on $$p^k$$ whose cost grows with $$k^2\sigma^2$$, and the interest is what pays for it. A rate that sounds competitive against exchange funding (0.1% a day, say) covers roughly ×1.5 on BTC and less on ETH; the leverages that break even at a bleed traders will tolerate are in the ×2 to ×4 range for crypto majors and higher only for low-volatility indices.

[Pool Parameters](pool-parameters.md) has the full recipe, tables by volatility and leverage, presets for common assets, and a backtest procedure. [LP Economics](../liquidity/lp-economics.md) has the derivation.
