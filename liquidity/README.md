---
description: The LP class and the Vault
---

# Liquidity

Liquidity in a Derion pool is a share class, not a role. The [LP class](lp-class.md) is the residual of the pool's two curves, tokenized as the pool's third ERC-1155 id, and anyone can hold it. There is no provider address, no registry, and no permission.

Two ways to hold it:

* **Directly.** Deposit reserve into a pool for its LP shares, through the same `transition()` path as every trade, and withdraw the same way. You choose the pool and the size.
* **Through the Vault.** Deposit reserve into the shared [Liquidity Vault](vault.md) for `VaultLP`. The Vault holds LP-class positions across many pools, keeps each pool's depth near its target, and is governed by its own depositors. It is one LP-class holder among any others and has no more power over a pool than anyone else.

### What the LP class earns

Every revenue stream of a pool accrues to it in-engine, as growth in reserve per share: the [funding](../protocol/funding-rate.md) traders pay (interest on both sides, premium from the crowded side), the [opening fee](../protocol/opening-fee.md), and the spread between the two oracle bounds on every trade executed while they diverge. Nothing is streamed to an address. The value is in the token.

### What it carries

The LP class is the counterparty. Its reserve is what the two power curves leave, so it is largest at the price center and shrinks as the price moves in either direction: a short straddle on $$p^k$$, with funding as its premium. [LP Economics](lp-economics.md) works out when that trade breaks even, and [Pool Parameters](../guide/pool-parameters.md) turns the answer into config values. Vault depositors carry the same exposure across the Vault's book, plus the policy risk of which pools it funds.

### Trader safety does not depend on liquidity

Payouts come from each pool's own reserve, capped below $$R$$ by the [curve](../protocol/pricing.md). A pool with an empty LP class is a pure trader-versus-trader market that deleverages instead of failing; a missed rebalance costs quality of service, never solvency. This is what lets liquidity be permissionless, rate-limited, and optional.
