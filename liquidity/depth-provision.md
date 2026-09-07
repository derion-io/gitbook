---
description: Keeping the dominant side at full leverage
---

# Depth Provision

A pool is solvent at any reserve level; depth is a quality-of-service matter. The Vault's job is to keep the dominant side of each served pool inside its full-leverage regime, below the [inflection point](../protocol/pricing.md), so traders on it see compounding $$k$$ over long moves rather than a decaying asymptote. It does this by holding [LP-class](lp-class.md) positions, which pad both curves at once.

## rebalance()

`rebalance(pool, index, rate, helper)` is permissionless and works on any pool the active [strategy](governance.md) covers. `index` locates the pool's record in the strategy; a wrong index reverts, so the call can never touch the wrong pool. It:

1. **Pokes** the pool, accruing funding and flushing the protocol cut.
2. **Computes the target** from the post-funding side reserves and the record's headroom $$h$$:

$$
R_t = 2\,\max(r_A, r_B)\,(1 + h)
$$

The inflection is $$R = 2\max(r_A, r_B)$$, where the dominant side sits exactly at $$R/2$$. Headroom pushes the target that fraction deeper, parking the dominant side at $$R / (2(1+h))$$, so it keeps full leverage until it has grown by a factor $$(1+h)$$. Zero headroom targets the inflection itself.

3. **Moves toward it.** If $$R_t > R$$ the Vault opens LP shares with idle reserve; if $$R_t < R$$ it closes part of its own position. The direction is never the caller's. The pool state picks it.

### The caller's size

`rate` is a fraction of the drift $$|R_t - R|$$ in $$(0, 1.5]$$: short of the target, exactly onto it, or past it by up to half the drift. The overshoot allowance exists because a close's first-order sizing and adverse-bound floor land short of what was asked, so aiming past the target is how one call finishes a ratchet instead of leaving a tail, or banks a buffer against the next drift. An ask above 1.5 is clamped, not reverted, because the pool moves between the worker's read and inclusion. Zero is a no-op.

### The caps

On top of the rate, two caps hold regardless of what the caller asks.

**Opens are ramp-limited against the Vault's own idle.** Per call, at most

$$
\text{idle}\cdot\left(1 - 2^{-\,\Delta t\,/\,T_{1/2}}\right)
$$

may flow into a pool, where $$T_{1/2}$$ is the record's ramp half-life and $$\Delta t$$ the time since the pool's last executed rebalance. The base is the Vault's idle, which a pool cannot inflate; a pool's reported reserves choose only the direction and the honest limit. Because each cap applies to the then-current idle, the bound is path-independent: over any window $$T$$, however many times the worker fires, at most $$\text{idle}\cdot(1 - 2^{-T/T_{1/2}})$$ of the idle reaches one pool. The Vault clamps every strategy's half-life up to its immutable `MIN_RAMP_HL`, so a strategy author cannot collapse the window.

**Closes are floored at the inflection.** A defund may pull back up to the caller's ask in one step, not ramped (a ramp on the exit would only trap the Vault in a pool it is trying to leave), but never below $$R = 2\max(r_A, r_B)$$: it cannot push the dominant side out of full leverage however the caller sizes it. It is also capped at the Vault's own position and floored at the pool's adverse-bound quote, so nothing beyond the Vault's own book is touched.

### The clock

The ramp's $$\Delta t$$ runs from the pool's last *executed* rebalance. A call that trades nothing (a no-op, a dust-skipped open) does not advance the clock; otherwise a permissionless caller could shrink the next fire's allowance and freeze a small pool. A pool that has never traded under the Vault, one freshly voted into the active set, anchors on the strategy's promotion time instead, so its first rebalance starts at $$\Delta t \approx 0$$ and can deploy almost nothing until it ages in. A pool wound down by `defund` has its clock cleared for the same reason: if it is voted back in it starts throttled again.

### Trade floors

`helper` is the caller's solver and is untrusted. Every Vault leg is floored at the pool's own adverse-bound quote, the same mark NAV uses, less a hair of rounding slack, so any solver can only match or beat the marked outcome. The pool's LP floor additionally stops a malicious solver from leaking a Vault deposit into a side.

## defund() and poke()

`defund(pool, strategy, gapIndex, helper)` winds down a pool that `strategy` does **not** fund, where `strategy` is either the active one or a candidate holding a strict majority. `gapIndex` is a non-membership proof: the pool's sorted insertion point in the blob, whose neighbours strictly bracket it, something a funded pool can never satisfy. The call closes the Vault's whole position there. Funding is gated by the strategy; defunding is always open, so a dropped pool can always be exited, and the candidate path is what lets an incoming majority clear the pools it drops before it can be promoted ([Strategy Governance](governance.md)).

`poke(pool)` and `pokeAll()` refresh funding on one pool or on every pool in the active set. All of these accept an optional `oracleData` blob for pools on pull oracles ([Price Oracle](../protocol/oracle/README.md)); one multi-feed blob serves every touch in the call.

## Headroom, economically

Headroom is a full-leverage buffer. At target the dominant side keeps compounding $$k$$ through a price move of

$$
\Delta_{\text{full}} = (1 + h)^{1/k} - 1
$$

before it reaches its inflection.

| k \ h | 5% | 10% | 20% | 30% | 50% | 100% |
| --- | --- | --- | --- | --- | --- | --- |
| 2 | 2.5% | 4.9% | 9.5% | 14.0% | 22.5% | 41.4% |
| 4 | 1.2% | 2.4% | 4.7% | 6.8% | 10.7% | 18.9% |
| 8 | 0.6% | 1.2% | 2.3% | 3.3% | 5.2% | 9.1% |
| 16 | 0.3% | 0.6% | 1.1% | 1.7% | 2.6% | 4.4% |

Read against daily volatility: a ×16 BTC pool with 10% headroom is at full leverage only inside ±0.6%, a fifth of a normal day. Past that the winner is on the asymptotic branch and its marginal leverage decays.

What headroom does for the LP class is bound the exposure per pool per refill cycle, since the winner can never claim more than $$R$$: at target, a matched book leaves $$r_C = 2rh$$ at risk and a one-sided one $$r_A(1 + 2h)$$. Imbalance, not headroom, is what exposes the LP, and the [premium](../protocol/funding-rate.md) is what pays for it.

The cap is a deferral, not a discount. A saturated winner's linear pay-off exceeds what it can claim; refilling the pool raises $$R$$ and hands over the difference. A pool refilled to the same headroom after every move pays the uncapped convexity over time, one refill late. The saving is real only on paths where the winner closes while still deleveraged or the price reverts before the refill lands. So headroom is a quality-of-service choice with a risk budget as its ceiling, and the ramp is the speed limit on the refill transfer. [LP Economics](lp-economics.md) has the derivation and [Pool Parameters](../guide/pool-parameters.md) the sizing.
