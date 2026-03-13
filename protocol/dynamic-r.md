---
description: Log-Symmetric Ratchet Rule for Linear-k Leverage
---

# Dynamic R Formula

## Notation

| Symbol | Meaning | Defined in |
|---|---|---|
| k | Leverage parameter | [Pay-off Curve](pricing.md) |
| x | Normalized price (= spot / MARK) | [Pay-off Curve](pricing.md) |
| α, β | Long and short coefficients | [Pay-off Curve](pricing.md) |
| R | Total reserve | [Pay-off Curve](pricing.md) |
| rA, rB | Long and short reserve values: rA = Φ(x), rB = Ψ(x) | [Pay-off Curve](pricing.md) |
| w | Virtual power-branch value: α·xᵏ | This page |
| T₀ | Base doubling time (pool parameter) | This page |
| T_d | Effective doubling time: k · T₀ | This page |
| R_eff | Effective R after time-decay ramping | This page |
| Φ, Ψ | Long and short payoff functions | [Pay-off Curve](pricing.md) |

---

## Context

Derion pools use asymptotic power curves where each side's payoff has two regimes separated by an **inflection point** at `rA = R/2`:

- **Power regime** (`rA < R/2`): `Φ(x) = α·xᵏ` — compound k leverage, convex curve
- **Asymptotic regime** (`rA > R/2`): `Φ(x) = R - R²/(4α·xᵏ)` — sub-linear leverage, concave curve

The inflection point is the only location where instantaneous leverage equals exactly k.

---

## Objective

For the dominant side of the pool, achieve **linear k leverage** over long price moves — meaning the integrated payoff across many ratchet cycles converges to `k × (price move)`, the same as a CEX perpetual.

Simultaneously, preserve **compound k downside** — when price moves against the dominant side, the full power curve applies.

---

## Why Log-Price Space Is The Natural Coordinate

The two branches have **unequal curvature in reserve space**:

```
Power branch:     d²Φ/dx²  =  +k(k-1)·α·x^(k-2)
Asymptotic branch: d²Φ/dx²  =  -k(k+1)·α·x^(k-2)

Curvature ratio = (k+1)/(k-1)   [always > 1]
```

Equal steps in reserve space produce asymmetric payoffs — the asymptotic branch always curves more steeply, causing systematic undershoot of linear k.

In log-price space (`u = ln x`), the curvatures are:

```
Power branch:     d²Φ/du²  =  +k²·α·xᵏ  →  at inflection: +k²·R/2
Asymptotic branch: d²Φ/du²  =  -(k²)·(R - Φ)  →  at inflection: -k²·R/2
```

**Exactly equal and opposite** at the inflection point, for any k. The two branches are perfect mirrors in log-price space. This is the natural coordinate system for the ratchet rule.

---

## The Ratchet Rule

### Symmetry Condition

For the linear-k payoff to be achieved over a full cycle, the position must spend equal log-price distance on each side of the inflection point:

```
ln(x_swap / x_inf_old) = ln(x_inf_new / x_swap)
```

That is, `x_swap` is the **geometric mean** of the old and new inflection prices:

```
x_inf_new = x_swap² / x_inf_old
```

### Deriving R_new

Using the inflection price formula `x_inf = (R / 2α)^(1/k)`:

```
x_inf_old = (R_old / 2α)^(1/k)
x_inf_new = (R_new / 2α)^(1/k)
```

Substituting into the symmetry condition:

```
x_inf_new = x_swap² / x_inf_old

(R_new / 2α)^(1/k) = x_swap² / (R_old / 2α)^(1/k)
```

Raising both sides to the power k:

```
R_new / 2α = x_swap^(2k) / (R_old / 2α)

R_new = 4α² · x_swap^(2k) / R_old
```

### Compact Form

Define `w = α · x_swap^k` — the **virtual power-branch value** at the swap price (what rA would be if it were still on the power branch):

```
┌─────────────────────────────┐
│                             │
│   R_new  =  4w² / R_old     │
│                             │
└─────────────────────────────┘
```

Equivalently:

```
√(R_new · R_old) = 2w
```

The geometric mean of old and new R equals twice the virtual power-branch value at the swap price.

### Ratchet Trigger

R is updated on every swap, with different rules for each direction:

```
w = α · x_swap^k          // virtual power-branch value at current price

if w > R_old/2:           // dominant side in asymptotic regime
    R_new = 4w² / R_old   // log-symmetric upward ratchet
    R_new = min(R_new, balanceOf(pool))   // hard cap at available balance

if w < R_old/2:           // dominant side in power regime
    R_new = 2w            // simple downward follow
    R_new = max(R_new, 2·rA, 2·rB)       // floor: neither side above inflection
```

#### Why The Two Rules Differ

**Upward** — the dominant side is in the asymptotic regime. Curvature is concave and payoff is sub-linear-k. The log-symmetric formula `4w²/R_old` positions the next power phase to exactly cancel the asymptotic undershoot. Curvature correction is required.

**Downward** — the dominant side is already in the power regime. Payoff is compound-k with no distortion. There is no curvature error to correct. R simply follows the dominant side down to `2w`, keeping it near inflection ready for the next upward move. No geometric correction needed — direct tracking is exact.

This asymmetry also governs dormant buffer flow: upward ratchets **consume** dormant capital to extend the linear-k runway; downward adjustments **reclaim** it as the market contracts.

---

## Validity Constraint

The floor `R_new ≥ max(2·rA, 2·rB)` on the downward rule ensures both sides remain at or below their inflection points at all times. This directly implies:

```
rA ≤ R_new/2   and   rB ≤ R_new/2
→  rA · rB ≤ R_new²/4
→  α·xᵏ · β·x⁻ᵏ ≤ R_new²/4
→  4αβ ≤ R_new²  ✅
```

For the upward rule, `R_new = 4w²/R_old > R_old`, so R only grows — validity is trivially preserved. The floor on the downward rule does the same work in the contracting direction.

---

## Position After Ratchet

After ratchet, the dominant side sits at:

```
rA_new / (R_new/2) = w / (2w²/R_old) = R_old / (2w)
```

Since the ratchet fires only when `w > R_old/2`, we have `R_old/(2w) < 1` — the position is always **strictly below the new inflection point**. The log-distance below the new inflection equals the log-distance above the old inflection at swap time.

---

## Dormant Buffer

Each swap adjusts the [dormant buffer](liquidity-provision.md):

- **Upward** (`w > R_old/2`): `R_new = 4w²/R_old > R_old` — buffer consumed, LP deploys capital to extend linear-k runway
- **Downward** (`w < R_old/2`): `R_new = 2w < R_old` — buffer reclaimed, LP recovers capital as dominant side retreats
- **Floor** `max(2·rA, 2·rB)` limits downward reclaim — a minimum R always covers both sides' current inflection points

---

## Rate-Limited Ramping

The ratchet formula computes an ideal `R_target` assuming honest price discovery. Without rate-limiting, two classes of adversaries can exploit instant R adjustment:

1. **Market manipulation.** An attacker spikes the price in a single block to inflate R, then dumps — extracting value from the dormant buffer before the market corrects.

2. **Insider trading.** An informed trader who knows a large price move is imminent opens a position just before the move. With instant R adjustment, the ratchet immediately expands R to accommodate the move, granting the insider full linear-k payoff on the entire swing. The LP bears the cost — the dormant buffer is consumed to fund the insider's amplified gain.

By rate-limiting R, the insider's edge is capped: even if they time the entry perfectly, R_eff only ramps slowly toward R_target. The insider's position spends most of the price move in the asymptotic regime (sub-linear-k payoff), limiting their profit to what an uninformed trader would have earned over the same ramp period. The LP's dormant buffer is consumed gradually rather than all at once.

To defend against both, R does not jump to its target instantly. Instead, R_eff moves toward R_target at a **maximum rate proportional to its own value** — a constant velocity in log space.

### Notation

| Symbol | Meaning |
|---|---|
| R_target | Ratchet-formula output (the ideal R) |
| R_eff | Effective R used by the pool |
| Δt | Time elapsed since last R update |
| T_d | Doubling time — time for R_eff to double (or halve) at max rate |

### Mechanism

The rate is derived from the doubling time: `μ = ln(2) / T_d`.

On each swap, after computing `R_target` from the [ratchet rule](#ratchet-trigger):

```
log_step = ln(R_target) − ln(R_eff)
max_step = ln(2) / T_d · Δt

if |log_step| ≤ max_step:
    R_eff = R_target                                    // within budget → apply fully

else:
    R_eff = R_eff · exp(sign(log_step) · max_step)      // clamp to max rate
```

The pool always uses `R_eff` — never `R_target` directly — for all payoff calculations, reserve splits, and validity checks.

### Example

With `T_d = 7 hours` (R can at most double in 7 hours):

If the ratchet formula wants R to double but only 1 hour has passed:

```
max_step = ln(2) / 7 · 1 ≈ 0.099      // ~10% in log space per hour
R_eff moves from R to R · e^0.099 ≈ R · 1.104
```

R_eff grows by ~10.4% per hour, reaching R_target after exactly `T_d = 7 hours`.

### Why Constant Rate In Log Space

**Predictable convergence time.** The time to reach any R_target is `|ln(R_target/R_eff)| · T_d / ln(2)` — deterministic and proportional to the log-distance. A 2× change always takes exactly T_d.

**Scale-invariant.** The rate cap is proportional to R_eff itself, so small and large pools have equivalent protection.

**Log-space consistency.** Operating in log-R space is consistent with the ratchet rule's log-price symmetry — a 2× increase and a 2× decrease in R require the same ramp time.

**Linear convergence.** Unlike exponential decay (which slows as it approaches the target), constant-rate ramping closes the gap at a steady pace. R_eff reaches R_target in finite time and stays there — no asymptotic tail.

### Properties

**Flash-loan resistance.** A single-block manipulation (Δt ≈ 2s, T_d = 7hr) can move R_eff by at most `ln(2)·2/25200 ≈ 0.005%` — negligible regardless of how extreme the price spike is.

**Organic moves pass through.** A sustained price trend accumulates ramp budget linearly with time. Real market moves reach R_target within T_d while single-block attacks are throttled by a factor of >10,000×.

**No queuing or state bloat.** The mechanism only stores `R_eff` and `t_last` — no target queue, no ramp schedules. Each swap simply computes how far R_eff is allowed to move given elapsed time.

**Preserves ratchet correctness.** Once R_eff reaches R_target (which it does in finite time for any sustained price level), all ratchet properties — log-symmetry, linear-k payoff, validity constraints — hold exactly as derived above.

### Scaling With K

Attacker profit from an R change scales linearly with k: a small price manipulation δ yields position gain proportional to `k · δ`. To keep the attacker's profit rate constant across leverage levels, the doubling time must scale linearly with k:

```
T_d = k · T₀
```

where T₀ is a **base doubling time** — a single pool parameter.

This scaling is also consistent from the recovery side: a price doubling requires R to grow by `2^(2k)`, taking time `2k · T_d` to fully accommodate. Recovery time scales linearly with k in both directions.

### Calibrating T₀

With `T_d = k · T₀`, the effective protection for a k=5 pool:

| T₀ | T_d (k=5) | 1-block attack (2s) | 1-min attack | 1-hr insider edge |
|---|---|---|---|---|
| 15 min | 1.25 hr | ~0.03% | ~1.8% | ~56% |
| 1 hr | 5 hr | ~0.008% | ~0.5% | ~15% |
| 7 hr | 35 hr | ~0.001% | ~0.07% | ~2% |

Adaptation time for a price doubling (`2k · T_d`):

| T₀ | k=2 | k=5 | k=10 |
|---|---|---|---|
| 1 hr | 4 hr | 10 hr | 20 hr |

A practical default: **T₀ = 1 hour**, giving:

- k=2: T_d = 2 hr — R doubles in 2 hr, adapts to 2× price in ~8 hr
- k=5: T_d = 5 hr — R doubles in 5 hr, adapts to 2× price in ~20 hr
- k=10: T_d = 10 hr — R doubles in 10 hr, adapts to 2× price in ~40 hr

---

## Behavior Across Lag Regimes

In practice, swaps do not fire at the exact inflection point — there is always some lag. This section analyzes how that lag accumulates over many ratchet cycles.

- **ε** — log-price distance below inflection immediately after a ratchet (the "headroom")
- **δ** — additional log-price drift into the asymptotic regime before the next swap fires (the "lag")
- **N** — number of ratchet cycles

| Effect | Order | Behavior |
|---|---|---|
| First-order lag error | Zero | Reflect rule absorbs δ into next cycle's ε — cancels exactly |
| Per-cycle curvature drag | O(εδ/R) | Small when oscillations are tight |
| Amplitude growth | ε → ε + δ per cycle | Inflates if lag is systematic |
| Cumulative drag (reserve-space rule) | O(N²δ²/R) | Grows quadratically with cycles |
| Cumulative drag (log-price rule) | O(Nδ²/R) | Linear — first-order cancellation holds for any lag magnitude |

The log-price formula eliminates the quadratic error term. For any lag size, the first-order cancellation holds because the curvature symmetry is exact at inflection in log-price coordinates. This is why the [ratchet rule](#the-ratchet-rule) operates in log-price space rather than reserve space.

---

## Comparison With Simpler Rules

| Rule | Up formula | Down formula | Log-price symmetric | Works for large lag |
|---|---|---|---|---|
| No ratchet | R fixed | R fixed | ❌ | ❌ dominant side stranded in asymptotic |
| Exact inflection | `R_new = 2·rA` | `R_new = 2·rA` | ❌ | ❌ ε collapses to 0, no symmetry |
| Reserve-space reflect | `R_old + 2ε` | `R_old - 2ε` | ❌ | ❌ O(N²δ²/R) cumulative drag |
| **This design** | **`4w²/R_old`** | **`2w`** | **✅ up exact** | **✅ O(Nδ²/R) only** |

---

## Summary

The ratchet formula `R_new = 4w²/R_old` is the unique rule that:

1. Keeps the dominant side oscillating symmetrically around the inflection point in log-price space
2. Achieves linear k leverage over long price moves, not compounding k
3. Preserves compound k downside when price moves against the dominant side
4. Remains self-consistent for any swap frequency — liquid or illiquid pools
5. Never violates pool validity constraints
6. Uses asymmetric rules for R adjustment — log-symmetric `4w²/R_old` on the way up (curvature correction required), simple `2w` on the way down (power regime, direct tracking is exact)
7. Rate-limits R changes via [rate-limited ramping](#rate-limited-ramping) — R_eff moves toward R_target with doubling time `T_d = k · T₀`, scaling protection with leverage while letting sustained price moves converge in finite time

{% hint style="info" %}
**See also:** [Dynamic LP Strategy](dynamic-lp.md) derives the break-even interest rate for an LP that implements this ratchet rule, and compares it against CEX perpetual funding rates.
{% endhint %}
