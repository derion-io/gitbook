---
description: Keeping the dominant side at full leverage
---

# Depth Provision

A pool is solvent at any reserve level — depth is a quality-of-service problem, not a safety one. The Vault's job is to keep the dominant side of each served pool near its [inflection point](../protocol/pricing.md), where instantaneous leverage equals exactly $$k$$. Pinned there, the dominant side experiences approximately **linear k leverage** over long price moves — the same as a CEX perpetual — while keeping the compound-k pay-off when the market turns against it.

## rebalance()

`rebalance(pool, index)` is permissionless and works on any pool the active [strategy](governance.md) covers (`index` locates the pool's record in the strategy; a wrong index reverts — it can never touch the wrong pool). It:

1. **Pokes** the pool (`sync()`), accruing and flushing pending funding.
2. **Rotates** to the minor side — closing any stale wrong-side position, so the Vault holds exactly one counterparty side per pool.
3. **Funds** the minor side toward the rate-limited inflection target, paying fresh reserve into the pool through the ordinary `transition()` path.

The first touch of a pool only anchors the ramp clock; movement begins from the next touch.

## The ratchet target

Where should $$R$$ be after a rebalance? Define $$w = \alpha x^k$$, the virtual power-branch value of the dominant side at the current price (what $$r_A$$ would be if it were still on the power branch). The target rule is asymmetric:

```
if w > R/2:               // dominant side saturated (asymptotic branch)
    R_target = 4w² / R    // log-symmetric upward ratchet

if w < R/2:               // dominant side on the power branch
    R_target = 2w         // simple downward follow
    R_target = max(R_target, 2·rA, 2·rB)   // keep both sides at or below inflection
```

The upward formula is chosen so the position spends equal log-price distance on each side of the inflection point — equivalently, $$\sqrt{R_{\text{target}} \cdot R_{\text{old}}} = 2w$$. Log-price space is the natural coordinate because the two branches of the curve have exactly equal and opposite curvature there ($$\pm k^2 R/2$$ at the inflection, for any $$k$$): a log-symmetric oscillation around the inflection makes the power branch's overshoot cancel the asymptotic branch's undershoot to first order, so the integrated pay-off converges to linear $$k$$ instead of accumulating drift. In reserve-space coordinates the same cancellation fails and the tracking error grows quadratically with the number of cycles.

The downward rule needs no correction: on the power branch the pay-off is undistorted compound-k, so $$R$$ simply follows the dominant side down, staying ready for the next upward move and reclaiming idle capital for the Vault as the market contracts.

## Rate-limited ramping

The ratchet output is a target, not a jump. Without a rate limit, two kinds of adversary could exploit instant $$R$$ adjustment: a manipulator spiking the oracle price for one block to inflate $$R$$ and dump into the added depth, and an insider opening just before a known move to harvest full linear-k pay-off on the entire swing at depositors' expense.

So the effective reserve moves toward the target at a constant velocity in log space, set by the pool's **doubling time** $$T_d$$ from its strategy record:

```
log_step = ln(R_target) − ln(R_eff)
max_step = ln(2) / T_d · Δt

R_eff moves by min(|log_step|, max_step) in log space
```

Properties of the constant log-rate:

* **Flash-loan resistance.** A single-block manipulation (Δt ≈ seconds against a T\_d of hours) moves $$R$$ by a negligible fraction, no matter how extreme the price spike.
* **Organic moves pass through.** A sustained trend accumulates ramp budget linearly with time and reaches the target within one doubling time per 2× of change — deterministic, with no asymptotic tail.
* **Scale-invariant.** The cap is proportional to the reserve itself, so small and large pools get equivalent protection.
* **Insider edge capped.** A perfectly-timed entry still spends most of the move on the asymptotic branch, earning little more than an uninformed trader over the same ramp period.

An attacker's profit from an $$R$$ change scales linearly with $$k$$, so the doubling time should scale with leverage too — $$T_d = k \cdot T_0$$ for a base doubling time $$T_0$$ — keeping the profit rate of an attack constant across leverage levels. The doubling time is set per pool in the [strategy](governance.md).

## defund() and poke()

`defund(pool, gapIndex)` winds down a pool the active strategy does **not** cover — for example, one dropped on a strategy change. The `gapIndex` is a non-membership proof (the pool's sorted insertion point, whose neighbors strictly bracket it — something no funded pool can satisfy), and the call closes whatever the Vault still holds there. Funding is gated by the strategy; **defunding is always open**, so a dropped pool can always be exited.

`poke(pool)` and `pokeAll()` refresh funding: they `sync()` the pool(s), flushing each pending outbox to the Vault.

{% hint style="info" %}
[LP Economics](lp-economics.md) derives the break-even funding rate for the Vault under this pinning strategy and compares it with CEX perpetual funding levels.
{% endhint %}
