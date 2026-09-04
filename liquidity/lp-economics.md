---
description: What the LP class earns, what it costs, and when it breaks even
---

# LP Economics

This page works out the economics of holding the [LP class](lp-class.md): what the position is, what it bleeds as the price moves, and the condition under which funding covers that bleed. The short version: the LP is short a straddle on $$p^k$$. The cost of that straddle grows with the square of the leverage and the square of the asset's volatility, and the only things that pay for it are interest, premium, and the opening fee. Everything below is a restatement of that sentence. [Pool Parameters](../guide/pool-parameters.md) turns it into config values.

## 1. What the LP owns

The LP class has no curve of its own. It owns whatever the two power curves leave:

$$
r_C = R - \rho(\alpha x^{k}, R) - \rho(\beta x^{-k}, R)
$$

Below the inflection the sides are pure power pay-offs and $$r_C$$ does not depend on $$R$$ at all. Over a price ratio $$x$$:

$$
\Delta r_C(x) = r_A\,(1 - x^{k}) + r_B\,(1 - x^{-k})
$$

For a matched book ($$r_A = r_B = r$$) this is $$-2r\,(\cosh(k \ln x) - 1)$$, negative for every $$x \ne 1$$. A balanced pool's LP loses on every price move in either direction and gets it back only if the price returns exactly to where it started. That is a short straddle, and the interest is its premium.

Per unit of time, the LP's P&L decomposes into six terms:

| Term | Sign | Source |
| --- | --- | --- |
| $$\varphi\,\lambda_I\,(r_A + r_B)$$ | + | interest: both sides pay rent on their reserve |
| $$\varphi\,\lambda_P\,\lvert r_A - r_B\rvert$$ | + | premium: the dominant side pays the gap down |
| $$f \cdot V_{\text{open}}$$ | + | opening fee on the gross reserve opened per unit time |
| $$G$$ | + | divergence gaps landed on $$r_C$$ (small; do not budget it) |
| $$\kappa_A r_A + \kappa_B r_B$$ | − | convexity: the winner grows faster than the loser shrinks |
| refill transfers | − | Vault refills paying out a saturated winner (§4) |

with $$\lambda_I = \ln 2 / \text{INTEREST\_HL}$$, $$\lambda_P = \ln 2 / \text{PREMIUM\_HL}$$, $$f$$ the fee as a fraction of the gross input, and $$\varphi = 1 - 1/\text{FEE\_RATE}$$ the LP's share of funding after the protocol cut (0.8 at the deployed `FEE_RATE = 5`). The first three are set by config. The convexity term is set by the asset.

## 2. The convexity cost

Take $$\ln x$$ over a horizon $$\tau$$ as drift $$m\tau$$ plus diffusion $$\sigma\sqrt{\tau}\,Z$$ plus a compound-Poisson jump component with intensity $$\nu$$ per year and log-jump size $$J$$. Then

$$
\mathbb{E}[x^{k}] = \exp\!\left(k m \tau + \tfrac{1}{2}k^2\sigma^2\tau + \nu\tau\,(\mathbb{E}[e^{kJ}] - 1)\right)
$$

and the LP bleeds, per unit of side reserve per year,

$$
\kappa_A = k m + \tfrac{1}{2}k^2\sigma^2 + \nu\,(\mathbb{E}[e^{kJ}] - 1), \qquad
\kappa_B = -k m + \tfrac{1}{2}k^2\sigma^2 + \nu\,(\mathbb{E}[e^{-kJ}] - 1)
$$

Three things to notice.

**The variance term is the whole story for a two-sided book.** Drift cancels between the sides when the book is matched, and for any drift $$\mathbb{E}[x^k] + \mathbb{E}[x^{-k}] \ge 2e^{k^2\sigma^2/2}$$. So $$\tfrac{1}{2}k^2\sigma^2$$ is a floor on the matched-book cost, not an approximation of it.

**It is quadratic in both $$k$$ and $$\sigma$$.** Doubling leverage quadruples the cost. Moving from BTC to a mid-cap alt (roughly double the vol) quadruples it again. Doing both is a 16× jump in what the interest has to cover.

**Jumps matter for stocks.** A symmetric scheduled jump of size $$\pm j$$ adds $$\nu\,(\cosh(kj) - 1)$$. For a mega-cap with four earnings a year and a 5% move, at $$k = 4$$ that is 0.08 per year on top of a diffusion term of 0.63, a 13% add. The diffusion term still dominates, but the jump term is what gets front-run, which is what the [opening fee](../protocol/opening-fee.md) is for.

**Flow toxicity.** The formula assumes traders open at random times. They do not: crypto flow chases momentum and stock flow clusters around events. Model this as a multiplier $$\xi \ge 1$$ on the variance term: 1 for a pool used as a hedge, 1.2–1.5 for majors with speculative flow, 1.5–2 for alts.

## 3. Break-even

For a matched book with zero drift, per unit of side reserve:

$$
\boxed{\;\varphi\,\lambda_I \;\ge\; \xi\,\tfrac{1}{2}k^2\sigma^2 + \nu\,(\cosh(kj) - 1)\;}
$$

Both sides scale with $$r_A + r_B$$, so the break-even does not depend on book size, on $$R$$, or (to first order) on headroom. It is a property of $$(k, \sigma, \text{INTEREST\_HL})$$ alone. Inverting it gives the interest half-life; a coverage quantile $$z$$ (how many daily sigmas the LP must absorb and still be flat) multiplies the variance term by $$z^2$$.

For a power-2 pool without jumps the memorable form is that the LP's net interest must be at least $$2\sigma^2$$ per year per unit of trader reserve. This is option theta in disguise: a power perpetual's convexity cost is proportional to variance, exactly as an at-the-money option's theta is $$\tfrac{1}{2}\Gamma\sigma^2 S^2$$.

What the interest costs the trader, as a daily bleed of position value:

| Half-life (days) | Daily bleed | Continuous rate / yr |
| --- | --- | --- |
| 7 | 9.4% | 3614% |
| 30 | 2.3% | 843% |
| 90 | 0.77% | 281% |
| 180 | 0.38% | 141% |
| 365 | 0.19% | 69% |
| 730 | 0.095% | 35% |

Perp funding is quoted per unit of notional. A position of value $$V$$ at power $$k$$ carries $$kV$$ of notional, so divide the bleed by $$k$$ to compare. In notional terms the inequality reads $$\varphi\lambda_I / k \ge \tfrac{1}{2}\xi k\sigma^2 + \ldots$$: the funding a power pay-off must charge per unit of notional grows linearly with $$k$$, the familiar power-perp result.

Break-even interest half-lives in days for a matched book at $$\varphi = 0.8$$, $$\xi = 1$$, no jumps, $$z = 1$$ (flat in expectation):

| k \ σ | 0.16 | 0.28 | 0.45 | 0.60 | 0.80 | 1.00 | 1.30 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | 3953 | 1291 | 500 | 281 | 158 | 101 | 60 |
| 3 | 1757 | 574 | 222 | 125 | 70 | 45 | 27 |
| 4 | 988 | 323 | 125 | 70 | 40 | 25 | 15 |
| 6 | 439 | 143 | 56 | 31 | 18 | 11 | 6.7 |
| 8 | 247 | 81 | 31 | 18 | 10 | 6.3 | 3.7 |
| 12 | 110 | 36 | 14 | 7.8 | 4.4 | 2.8 | 1.7 |
| 16 | 62 | 20 | 7.8 | 4.4 | 2.5 | 1.6 | 0.9 |
| 32 | 15 | 5.0 | 2.0 | 1.1 | 0.6 | 0.4 | 0.2 |

At $$z = 2$$ (flat on about 95% of days) divide by 4. A BTC ×16 pool would need a half-life near two days, a 27%/day bleed. There is no interest rate at which it works; the fix is a lower leverage.

### Comparison with exchange funding

Centralized perpetual funding runs around 0.01% per 8 hours (about 11% a year) in a neutral market and 0.1–0.3% a day in a crowded one. Derion is selling convexity without liquidation and is not competitive at the calm rate. At the crowded rate BTC supports roughly ×2–5 and ETH ×1–3. What makes the position workable is that high volatility and high funding demand arrive together: bull markets raise both the LP's cost and the traders' willingness to pay. Structurally the LP is short volatility, which is what a liquidity provider is.

What the trader buys for that funding is real. At the same exposure a linear perp carries a liquidation price and no convexity; the Derion position has positive gamma and no liquidation at any price. The LP's $$\tfrac{1}{2}k^2\sigma^2$$ is the fair price of writing it.

## 4. Headroom is a deferral

The Vault refills a pool to $$R_t = 2\max(r_A, r_B)(1 + h)$$ ([Depth Provision](depth-provision.md)). At target the residual is

$$
r_C = (1 + 2h)\max(r_A, r_B) - \min(r_A, r_B)
$$

($$2rh$$ for a matched book, $$r_A(1 + 2h)$$ for a one-sided one), and that is the most the LP can lose before the next refill, because the winner can never claim more than $$R$$. Imbalance, not headroom, is what exposes the LP.

The caveat is that the cap defers rather than saves. When the winner is saturated its linear pay-off $$q = \alpha x^k$$ exceeds what it can claim, $$\rho(q, R)$$, and a refill from $$R_0$$ to $$R_1$$ hands over $$\rho(q, R_1) - \rho(q, R_0)$$. The depositor (the Vault) pays it knowingly, priced at the class's dear bound with incumbents protected, but a pool refilled to the same headroom after every move pays the uncapped convexity over time, one refill late. Do not use the cap to shorten the interest half-life: with a working refill loop the long-run cost is the uncapped one.

## 5. A worked loss

A WBTC/USD₮0 ×16 pool with 10% headroom and funding switched off, two positions of 1 ETH long and 1 ETH short, and the price up 4.1% a day later. The long shows 1.875 ETH, the short 0.526 ETH, the LP's P&L −0.401 ETH.

Linear pay-offs at +4.1% are $$1.041^{16} = 1.902$$ and $$1.041^{-16} = 0.526$$. The short is exactly on its curve; the long is marked slightly below, consistent with each side quoted at its adverse basis under a small TWAP/spot divergence. Suppose the Vault had the pool at target when the positions opened: $$R = 2.2$$, $$r_C = 0.2$$. Through the move the long saturates, $$\rho(1.90, 2.2) = 1.563$$, the short falls to 0.526, and $$r_C = 0.111$$: a loss of 0.089 so far, with 0.34 ETH of deferred pay-off the long cannot yet claim. A rebalance then fires. The new target is $$2 \cdot 1.563 \cdot 1.1 = 3.44$$, a 1.24 ETH deposit of which 0.32 goes straight to the long. Cumulative loss 0.409. One more refill would take the long to its full 1.90 and the loss to 0.426, the uncapped $$(1.902 - 1) - (1 - 0.526)$$.

Interest collected over the day: none. To have netted that day the pool would have needed a half-life of about 2.2 days. There is no parameter fix at ×16 on BTC. The fix is $$k \le 4$$, or accepting the pool as a loss leader with a headroom small enough and a ramp slow enough that the loss per day stays near $$2rh$$.
