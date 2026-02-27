---
description: Dynamic LP Strategy at the Deleverage Point — Break-Even Analysis & Binance Funding Comparison
---

# Dynamic LP Strategy

## Overview

This document develops a **dynamic LP strategy** for Derion pools that keeps the larger side of (A, B) pinned at the deleverage point, effectively maintaining linear leverage. We derive the **break-even interest rate** for a BTC pool with no premium, then compare it against real-world **Binance perpetual funding rates**.

***

## 1. Protocol Recap

Derion is a fully on-chain power perpetuals AMM with three pool sides: **A** (Long), **B** (Short), and **C** (LP). The pool state is the tuple ⟨R, α, β⟩ where R is total reserve, α and β are the long and short coefficients.

### 1.1 Payoff Curves

For leverage parameter k and normalized price x (= spot / MARK):

**Long payoff:**

$$
\Phi(x) = \begin{cases} \alpha x^k & \text{if } x \le \left(\frac{R}{2\alpha}\right)^{1/k} \quad \text{(power branch, leverage = k)} \\ R - \frac{R^2}{4\alpha x^k} & \text{otherwise} \quad \text{(asymptotic branch, leverage → 0)} \end{cases}
$$

**Short payoff:**

$$
\Psi(x) = \begin{cases} \beta x^{-k} & \text{if } x \ge \left(\frac{2\beta}{R}\right)^{1/k} \quad \text{(power branch, leverage = −k)} \\ R - \frac{R^2 x^k}{4\beta} & \text{otherwise} \quad \text{(asymptotic branch, leverage → 0)} \end{cases}
$$

**LP (Counterparty Liquidity):**

$$
\Omega(x) = R - \Phi(x) - \Psi(x)
$$

### 1.2 Key Quantities

From the pool state, the reserves of each side are: `rA = Φ(x)`, `rB = Ψ(x)`, `rC = R − rA − rB`. The **deleverage prices** are the inflection points:

$$
\text{dgA} = \text{MARK} \times \left(\frac{R}{2\alpha}\right)^{1/k} \qquad \text{dgB} = \text{MARK} \times \left(\frac{2\beta}{R}\right)^{1/k}
$$

### 1.3 Interest & Premium

**Interest** decays both α and β by factor $$2^{-t/H}$$, transferring value from Long+Short → LP:

$$
\alpha' = \alpha \times 2^{-t/\text{INTEREST\_HL}}, \qquad \beta' = \beta \times 2^{-t/\text{INTEREST\_HL}}
$$

The effective interest rate charged to each side is:

$$
\text{side}[A].\text{interest} = \text{interestRate} \times \frac{K}{k_{\text{eff},A}}
$$

**Premium** transfers from the larger side to the smaller side + LP, pro-rata:

$$
\text{PR}_{\text{long}} = \text{PR}_{\max} \times \frac{r_A^2 - r_B^2}{R \times r_A}
$$

***

## 2. The Deleverage Point & Leverage Regimes

Each side of the curve has two regimes separated by the **inflection point** (= deleverage point):

| Regime | Long (Side A) | Short (Side B) |
| --- | --- | --- |
| **Power branch** (full leverage) | x < (R/2α)^{1/k} → leverage = k | x > (2β/R)^{1/k} → leverage = −k |
| **Asymptotic branch** (deleveraged) | x > (R/2α)^{1/k} → leverage → 0 | x < (2β/R)^{1/k} → leverage → 0 |

The effective leverage of side A at price x:

$$
k_{\text{eff}}(A) = \frac{x \cdot \Phi'(x)}{\Phi(x)}
$$

| Branch | k\_eff | rA |
| --- | --- | --- |
| Power | k (full) | < R/2 |
| At inflection | k (exact) | = R/2 |
| Asymptotic | kR/(4αx^k − R) → 0 | > R/2 |

{% hint style="info" %}
**Key Insight:** When a side crosses its deleverage point, its payoff value exceeds R/2 and its leverage compresses toward zero. The **larger** side of (A, B) is the one closest to or past its deleverage point. At exactly the deleverage point, that side's reserve = R/2 and leverage = k.
{% endhint %}

***

## 3. The Dynamic LP Strategy

**Goal:** Always keep the larger of (rA, rB) pinned at approximately the deleverage point — i.e., the larger side's reserve ≈ R/2. This ensures the dominant side stays at exactly linear leverage k.

### 3.1 Mechanism

1. **Monitor the pool state.** After each price move, check if max(rA, rB) has moved away from R/2.
2. **If price rises** (rA grows), rA may exceed R/2 and enter the asymptotic branch. The LP **adds liquidity** (increases R) so the deleverage point shifts up and rA returns to ≈ R/2.
3. **If price falls** (rB grows), the LP adjusts symmetrically to keep rB at its deleverage boundary.
4. **Net effect:** At all times, `max(rA, rB) ≈ R/2`, both sides maintain effective leverage ≈ k.

```
Price rises → rA > R/2 → LP adds R (adjusts α) → rA = R/2 restored
Price falls → rB > R/2 → LP adds R (adjusts β) → rB = R/2 restored
```

### 3.2 What "Linear Leverage" Means

At the deleverage point, the payoff function transitions from convex (power branch) to concave (asymptotic branch). At this exact point, the value is R/2 and the instantaneous leverage is k. For any further movement in the winning direction, leverage compresses. By always rebalancing to keep the dominant side at this point, the LP ensures:

The dominant side's PnL behaves approximately **linearly** in x (leverage ≈ k for small moves), not super-linearly (deep in power branch) and not sub-linearly (deep in asymptotic branch).

***

## 4. LP PnL Under This Strategy

### 4.1 LP Exposure at the Deleverage Point

Under the constraint max(rA, rB) = R/2, assume the Long side is dominant (rA ≈ R/2):

$$
r_A = \alpha x^k = R/2, \quad r_B = \beta x^{-k} \text{ (small)}, \quad r_C = R/2 - \beta x^{-k}
$$

### 4.2 LP Gamma

The LP's gamma at the deleverage point (from the power payoff second derivative):

$$
\Gamma_{\text{LP}} \approx -\frac{k(k-1) \cdot R}{2x^2}
$$

This is **negative gamma** — the LP loses from price volatility (impermanent loss).

### 4.3 Impermanent Loss Rate

For a GBM price process with volatility σ:

$$
\frac{dIL}{dt} = \frac{1}{2} \times |\Gamma_{\text{LP}}| \times \sigma^2 \times x^2 = \frac{k(k-1) \cdot R \cdot \sigma^2}{4}
$$

Per unit of LP capital (≈ R/2):

$$
\boxed{\text{IL rate per LP capital} = \frac{k(k-1) \cdot \sigma^2}{2}}
$$

***

## 5. Break-Even Interest Rate

With **no premium** (premium rate = 0), the only income for the LP is the interest rate.

### 5.1 Interest Income

The interest rate r decays rA and rB, transferring value to LP:

$$
\text{Income per unit R} = r \times \frac{r_A + r_B}{R}
$$

With the deleverage constraint (rA ≈ R/2, rB small), rA + rB ≈ R/2.

### 5.2 Break-Even Condition

Setting interest income ≥ IL cost:

$$
r \times \frac{1}{2} \ge \frac{k(k-1) \cdot \sigma^2}{4}
$$

$$
\boxed{r_{\text{break-even}} = \frac{k(k-1) \cdot \sigma^2}{2}}
$$

**For k = 2 (BTC pool):** This simplifies elegantly to:

$$
\boxed{r_{\text{break-even}} = \sigma^2}
$$

The break-even interest rate equals BTC's **variance**.

### 5.3 After Protocol Fee

Derion takes 1/5 of interest income to LP as protocol fee. The gross break-even becomes:

$$
r_{\text{gross}} = \frac{k(k-1) \cdot \sigma^2}{2 \times 0.8} = \frac{k(k-1) \cdot \sigma^2}{1.6}
$$

***

## 6. Numerical Results for BTC

### 6.1 Break-Even by Volatility

Using formula $$r = k(k-1)\sigma^2 / 2$$ (gross, before protocol fee):

| k | σ = 40% | σ = 50% | σ = 60% | σ = 70% | σ = 80% |
| --- | --- | --- | --- | --- | --- |
| **2** | **16%** | **25%** | **36%** | **49%** | **64%** |
| 4 | 96% | 150% | 216% | 294% | 384% |
| 8 | 448% | 700% | 1008% | 1372% | 1792% |

### 6.2 Converting to Derion's Half-Life

$$
\text{INTEREST\_HL} = \frac{\ln 2}{r / \text{seconds\_per\_year}} = \frac{0.693}{r / 31{,}536{,}000}
$$

| Scenario (k=2) | Annual Rate | Daily Rate | Half-Life |
| --- | --- | --- | --- |
| σ = 50% (low vol) | 25% | 0.068% | ~1,012 days |
| **σ = 60% (mid vol)** | **36%** | **0.099%** | **~703 days** |
| σ = 70% (high vol) | 49% | 0.134% | ~516 days |
| σ = 80% (very high) | 64% | 0.175% | ~395 days |

{% hint style="success" %}
**BTC Pool (k=2, σ=60%):** Break-even interest ≈ **36% annualized** (≈ 0.10% daily), half-life ≈ 703 days. After 20% protocol fee → pool needs ~45% gross, half-life ≈ 562 days.
{% endhint %}

***

## 7. Comparison with Binance Perp Funding

### 7.1 Binance BTC Perp Funding — The Benchmark

Binance charges BTC perpetual funding every 8 hours. The baseline (when perp ≈ spot) is 0.01% per 8h, annualizing to ~10.95%. In practice, funding varies dramatically:

| Market Regime | Typical 8h Rate | Annualized |
| --- | --- | --- |
| Baseline / neutral | 0.0100% | **~10.95%** |
| Mild bull (normal trending) | 0.01–0.03% | ~11–33% |
| Strong bull (2024 BTC run) | 0.03–0.06% | ~33–66% |
| Extreme euphoria (peaks) | 0.05–0.15% | ~55–165% |
| Bear / capitulation | −0.01 to 0.00% | ~−11 to 0% |

Key data points:

* BTC aggregate funding was **overwhelmingly positive in 2024**, only 26 days negative.
* OI-weighted funding has been **positive >85% of the time** over the past 2 years.
* Many exchanges use a base interest rate that skews default funding to ~10.95% annualized.
* Peak: OI-weighted average reached **109% annualized** on Feb 28, 2024.
* Long-run average: approximately **11–22% annualized**; in bull years like 2024, closer to **15–30%**.

### 7.2 Head-to-Head Comparison

| | Binance BTC Perp | Derion BTC LP (k=2) |
| --- | --- | --- |
| **Rate type** | Market-driven + 10.95% base | Pool-configured (INTEREST\_HL) |
| **Long-run average** | ~11–22% annualized | Needs σ² = 25–64% |
| **Bull market avg** | ~20–30% annualized | σ=50% → needs 25% |
| **Peak euphoria** | 55–165% annualized | σ=70% → needs 49% |

### 7.3 Visual Scale

```
0%        20%        40%        60%        80%       100%+
|----------|----------|----------|----------|----------|
██████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  Binance base (11%)
██████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░  Binance bull avg (25%)
████████████████████░░░░░░░░░░░░░░░░░░░░░░  Derion BE σ=50% (25%)
██████████████████████████░░░░░░░░░░░░░░░░  Derion BE σ=60% (36%)
████████████████████████████████████░░░░░░  Derion BE σ=70% (49%)
██████████████████████████████████████████  Binance peak 2024 (60%+)
```

### 7.4 Scenario Analysis

| Scenario | BTC σ | Derion Needs | Binance Pays | Verdict |
| --- | --- | --- | --- | --- |
| Low vol regime | 40% | 16% | ~11–15% | ⚠️ Tight / underwater |
| Normal vol | 50% | 25% | ~15–25% | ⚠️ Marginal |
| Elevated vol + bull | 60% | 36% | ~25–60% | ✅ Often covered |
| High vol + euphoria | 70%+ | 49%+ | ~50–100%+ | ✅ Typically covered |

### 7.5 Structural Differences

**Apples-to-apples on 2× leverage exposure:**

| | Binance (2× levered perp) | Derion (k=2 power perp) |
| --- | --- | --- |
| Funding per unit capital | 2 × 11% = 22% (baseline) | r (pool interest rate) |
| Liquidation risk | **Yes** (at ~50% adverse move) | **None** (asymptotic curve) |
| Gamma / convexity | Zero (linear payoff) | **Positive** (power payoff) |

Derion traders get **positive convexity** and **no liquidation** — valuable features that justify paying a higher funding rate. The LP bears the cost of that convexity, which is exactly the σ² term.

***

## 8. Key Observations

### 8.1 The Variance-Funding Relationship

For k=2, the break-even is simply **σ²**. This is not coincidental — it mirrors option theta. A power-2 perpetual's gamma cost is proportional to the variance, just as ATM option theta is ½Γσ²S². The Derion LP is essentially **writing a power perpetual**, and σ² is its theoretical fair premium.

Binance's funding rate is set by **market forces** (supply/demand of leverage) plus a fixed base, **not directly tied to realized volatility**. This creates a structural mismatch that can work in either direction.

### 8.2 High Vol and High Funding Are Correlated

The crucial nuance: **high funding rates and high volatility tend to occur together**. Bull markets produce both high σ (increasing LP costs) _and_ high funding (increasing LP income). Empirically during peak euphoria, funding can reach 100%+ annualized — well above σ² ≈ 49–64%. But during quiet periods (σ ≈ 40%), base funding of 11% falls short of the 16% break-even.

### 8.3 Premium Rate Closes the Gap

Our break-even formula assumes **zero premium**. In practice, under the dynamic strategy (rA ≈ R/2, rB small), the imbalance is large → premium income is significant. This additional income means the **actual break-even interest rate is lower than σ²**, potentially making the position profitable even at Binance's base rate of ~11%.

### 8.4 Higher k Pools

The break-even scales as k(k−1), making high-leverage pools require dramatically higher interest:

* k=4: needs 6σ² → ~216% at σ=60%
* k=8: needs 28σ² → ~1008% at σ=60%

Only very short-term positions are economical for traders in high-k pools.

***

## 9. Conclusion

{% hint style="success" %}
**Main Result:** For a Derion pool with leverage k, underlying volatility σ, and no premium, the LP break-even interest rate under the dynamic deleverage-point strategy is:

$$r_{\text{break-even}} = \frac{k(k-1) \cdot \sigma^2}{2}$$

For a **BTC pool with k=2** and typical σ ≈ 60%, this gives **r ≈ 36% annualized** (≈ 0.10% daily).
{% endhint %}

{% hint style="info" %}
**vs Binance:** The Derion LP needs roughly **2–3× Binance's base rate** but only about **1–1.5× the bull-market average**. Including premium income (material under the dynamic strategy) and the vol-funding correlation, the strategy is **viable in trending/active markets but marginal in quiet ones** — structurally equivalent to being short volatility, which is exactly what an LP position is.
{% endhint %}

### Practical Considerations

* **Rebalancing frequency:** Discrete rebalancing introduces tracking error. The break-even is a lower bound; actual required rates may be 10–30% higher.
* **Premium as buffer:** Under the dynamic strategy, persistent imbalance generates premium that can cover 30–50% of the IL cost.
* **Protocol fee:** 20% of interest to LP is taken as protocol fee — gross pool rate must be 1.25× the net break-even.
* **Gas costs:** On-chain rebalancing has transaction costs that reduce net LP returns.
* **Comparison to Squeeth:** For Opyn's Squeeth (k=2), the funding rate is approximately σ²/year. Derion's LP break-even at k=2 is also σ², confirming the mathematical equivalence of the gamma cost.
