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

For the upward rule, `R_new = 4v²/R_old > R_old`, so R only grows — validity is trivially preserved. The floor on the downward rule does the same work in the contracting direction.

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

{% hint style="info" %}
**See also:** [Dynamic LP Strategy](dynamic-lp.md) derives the break-even interest rate for an LP that implements this ratchet rule, and compares it against CEX perpetual funding rates.
{% endhint %}
