---
description: Choosing leverage, funding, fee, headroom, and ramp so the LP class breaks even
---

# Pool Parameters

How to pick the leverage, `INTEREST_HL`, `PREMIUM_HL`, `OPEN_RATE`, and the Vault's per-pool headroom and ramp for a given asset, so that the [LP class](../liquidity/lp-class.md) comes out flat or better across the price moves the asset actually produces. [LP Economics](../liquidity/lp-economics.md) derives the model this page applies: the LP is short a straddle on $$p^k$$, its cost grows with the square of leverage and the square of volatility, and interest, premium, and the opening fee are what pay for it.

The steps below are implemented in the parameter calculator that ships with the core repository (`deploy/params.html`; the repository is not public at the time of writing). It fetches the price history for a ticker, CoinGecko id, or on-chain pool address, measures the inputs, emits the config fields, and replays the engine over the history and over synthetic regimes.

## 0. Notation

| Symbol | Meaning |
| --- | --- |
| $$k$$ | pay-off power, the displayed leverage. Side value ∝ $$p^k$$ (long) or $$p^{-k}$$ (short) below the inflection. |
| `K` | the config field. Sqrt-price fetcher (Uniswap v3, Composite): `K = 2k`. Plain-price fetcher (Chainlink, stock tokens): `K = k`. Check which one the pool uses first. |
| $$x$$ | price ratio $$p_t/p_0$$ over the horizon considered |
| σ | annualized volatility of $$\ln p$$, per calendar year, as a decimal |
| $$\sigma_d$$ | one-day volatility. Crypto: $$\sigma/\sqrt{365}$$. Stocks: $$\sigma/\sqrt{252}$$. |
| $$\lambda_I = \ln 2 / \text{INTEREST\_HL}$$ | continuous interest rate |
| $$\lambda_P = \ln 2 / \text{PREMIUM\_HL}$$ | continuous premium rate on the reserve gap |
| $$f$$ | opening fee as a fraction of the gross input: $$f = 1 - \text{OPEN\_RATE}/2^{128}$$ |
| $$\varphi$$ | the LP's share of funding after the protocol cut: $$1 - 1/\text{FEE\_RATE}$$, 0.8 at the deployed `FEE_RATE = 5` |
| $$h$$ | Vault headroom (`headRoom / 2^16`); rebalance target $$R_t = 2\max(r_A, r_B)(1+h)$$ |
| $$T_{1/2}$$ | Vault per-pool ramp half-life; an open is capped at $$\text{idle}\,(1 - 2^{-\Delta t/T_{1/2}})$$ |
| $$z$$ | coverage quantile: how many daily sigmas the LP must absorb and still be flat |
| $$\xi$$ | flow-toxicity multiplier on variance (1 = random flow) |
| $$\nu, j$$ | scheduled-jump intensity per year and typical absolute log move |
| $$e$$ | the LP's target edge per unit of trader reserve per year, net of the convexity cost |

## 1. Leverage

Leverage is the first thing to fix and the only one with a hard external constraint: what traders will pay to hold a position. Quote that tolerance the way perp funding is quoted, per unit of *notional* per day, $$f_{\max}$$, so it can be set against exchange funding directly. A side at power $$k$$ carries $$k$$ units of notional per unit of value, so what it pays on its value is $$k$$ times the notional rate, and what the LP can collect per unit of trader reserve is $$\varphi k \lambda_n$$, rising with $$k$$. The convexity it has to cover rises with $$k^2$$. The leverages that work are therefore an interval, and $$k_{\max}$$ is its upper end:

$$
\lambda_n = -365 \ln(1 - f_{\max})
$$

$$
\xi\,\tfrac{1}{2}k^2\sigma^2 + \nu\,(\cosh(kj) - 1) + e \;\le\; \varphi\, k\, \lambda_n
$$

$$
k_{\max} = \frac{\varphi\lambda_n + \sqrt{(\varphi\lambda_n)^2 - 2\xi\sigma^2 e}}{\xi\sigma^2} \qquad \text{(no jumps; with jumps solve numerically)}
$$

At $$e = 0$$ this is $$k_{\max} = 2\varphi\lambda_n / (\xi\sigma^2)$$. Round *down* to the `K` grid: half-steps of $$k$$ on a sqrt-price fetcher, whole steps on a plain-price one.

Two conventions, one number. The tolerance above is on notional because that is how the market it competes with quotes carry. Every "bleed" figure elsewhere on this page is on *position value*, because that is what a holder sees disappear: the fraction of a 1 ETH position gone by tomorrow at an unchanged price. They convert by $$k$$: value bleed = $$k$$ × notional rate. Quoting per notional does not make a pool cheaper; it makes the comparison honest.

$$k_{\max}$$ at $$\varphi = 0.8$$, $$\xi = 1$$, $$e = 0$$ (break-even in expectation), tolerance as funding on notional:

| σ | 0.03%/day | 0.05%/day | 0.1%/day | 0.2%/day | 0.3%/day | 0.5%/day |
| --- | --- | --- | --- | --- | --- | --- |
| 0.16 (index ETF) | 6.8 | 11.4 | 22.8 | 45.7 | 68.5 | 114 |
| 0.28 (mega-cap) | 2.2 | 3.7 | 7.5 | 14.9 | 22.4 | 37.3 |
| 0.45 | 0.9 | 1.4 | 2.9 | 5.8 | 8.7 | 14.5 |
| 0.60 (BTC, TSLA) | 0.5 | 0.8 | 1.6 | 3.2 | 4.9 | 8.1 |
| 0.80 (ETH) | 0.3 | 0.5 | 0.9 | 1.8 | 2.7 | 4.6 |
| 1.00 (SOL) | 0.2 | 0.3 | 0.6 | 1.2 | 1.8 | 2.9 |
| 1.30 (mid-cap alt) | 0.1 | 0.2 | 0.3 | 0.7 | 1.0 | 1.7 |

For reference, 0.03%/day is the baseline funding of a centralized perp in a calm market and 0.1–0.3%/day is a crowded one. Derion sells convexity without liquidation, so it is not competitive at the calm rate, and the table says so. At the crowded rate BTC supports ×2–5 and ETH ×1–3. Low-volatility index ETFs support far more on this test alone, because the collectable interest scales with $$k$$ while the variance cost scales with $$k^2\sigma^2$$; for those the event bound (§4) and the headroom needed to stay at full leverage (§5) bind long before funding does. A positive edge target lowers $$k_{\max}$$ and also creates a *minimum* viable $$k$$, the lower root: with income proportional to $$k$$, a ×1 pool cannot earn a fixed edge at any tolerated rate. BTC ×16 needs the LP to receive about 2/yr of notional, roughly 0.55%/day, or 0.7%/day once the protocol cut is grossed up, which is 9–11% of position value per day. No framing changes that number.

If the product needs a *displayed* number higher than $$k_{\max}$$, the honest way to do it is a small headroom plus a slow ramp (§5, §6): the winner is deleveraged past a small move, and the LP's per-move loss is capped. Understand that this is a different product (leverage that decays as you win), and the refill dynamics in §5 mean the cap is a deferral, not a discount.

## 2. `INTEREST_HL`

Invert the break-even inequality:

$$
\text{INTEREST\_HL} \;\le\; \frac{\varphi \ln 2}{\xi z^2 \tfrac{1}{2}k^2\sigma^2 + \nu\,(\cosh(kj) - 1)} \qquad \text{[years]}
$$

or, without jumps, the memorable form $$\text{HL} \le 2 \ln 2\, \varphi / (z^2 k^2 \sigma^2)$$.

For a single day the exact form is $$\varphi\,(1 - 2^{-1/\text{HL}_d}) \ge \cosh(k z \sigma_d) - 1$$, which matters once $$k z \sigma_d$$ is above about 0.3; below that the two agree within a few percent.

Break-even `INTEREST_HL` in days for a matched book at $$\varphi = 0.8$$, $$\xi = 1$$, no jumps, $$z = 1$$:

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

$$z = 2$$ (flat on a 2σ day, about 95% of days): divide by 4.

What those half-lives cost the trader, as a fraction of position value (divide by $$k$$ for the notional-equivalent that compares with perp funding):

| HL (days) | Daily bleed | Continuous rate / yr |
| --- | --- | --- |
| 3 | 20.6% | 8433% |
| 7 | 9.4% | 3614% |
| 14 | 4.8% | 1807% |
| 30 | 2.3% | 843% |
| 60 | 1.15% | 422% |
| 90 | 0.77% | 281% |
| 180 | 0.38% | 141% |
| 365 | 0.19% | 69% |
| 730 | 0.095% | 35% |

**Which $$z$$ to use.** $$z = 1$$ means the LP is flat on an average day and negative on roughly a third of days, with the losing days fatter than the winning ones. $$z = 1.5$$ makes the expected daily edge about 56% of the interest collected, enough for most weeks to close positive. $$z = 2$$ is conservative and roughly halves the leverage you can offer. The presets in §8 use $$z = 1.5$$.

The same margin can be stated the way an LP thinks about it, as a target edge $$e$$ per unit of trader reserve per year, net of the convexity cost: $$e = (z^2 - 1)\,\xi\,\tfrac{1}{2}k^2\sigma^2$$, and then $$\text{INTEREST\_HL} = \varphi \ln 2 / (\xi\tfrac{1}{2}k^2\sigma^2 + \nu(\cosh(kj) - 1) + e)$$. Divided by the LP's residual at the ratchet target, $$r_C \approx (1 + 2h)\max - \min$$ per unit of book, it is the return on the capital the LP actually has at risk in the pool. The calculator takes $$e$$ as its input and reports the implied $$z$$ and day coverage.

## 3. `PREMIUM_HL`

Interest pays for variance. The premium pays for *direction*. When the book is one-sided the LP is the counterparty to a net leveraged position; if the price trends at $$m$$ the LP loses $$k m |r_A - r_B|$$ per year with no offsetting variance income. The premium charges the dominant side $$\lambda_P |r_A - r_B|$$ and pays it entirely to $$r_C$$, so the LP is compensated exactly on the exposure it carries:

$$
\varphi\lambda_P \;\ge\; k\, m_{\text{ref}} \qquad\Longrightarrow\qquad \text{PREMIUM\_HL} \;\le\; \frac{\varphi \ln 2}{k\, m_{\text{ref}}}
$$

Budgeting for a realized drift you can measure is circular; a persistent trend is what an informed dominant side is betting on. A defensible reference is a Sharpe-$$S$$ trend, $$m_{\text{ref}} = S\sigma$$, with $$S = 1$$ for a pool that should survive a strong directional regime and $$S = 0.5$$ for one where you accept some directional risk to stay cheap:

$$
\text{PREMIUM\_HL} \;\le\; \frac{\varphi \ln 2}{k\, S\, \sigma}
$$

`PREMIUM_HL` in days, $$\varphi = 0.8$$, $$S = 1$$:

| k \ σ | 0.16 | 0.28 | 0.45 | 0.60 | 0.80 | 1.00 | 1.30 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | 633 | 361 | 225 | 169 | 127 | 101 | 78 |
| 3 | 422 | 241 | 150 | 112 | 84 | 68 | 52 |
| 4 | 316 | 181 | 112 | 84 | 63 | 51 | 39 |
| 6 | 211 | 121 | 75 | 56 | 42 | 34 | 26 |
| 8 | 158 | 90 | 56 | 42 | 32 | 25 | 20 |
| 16 | 79 | 45 | 28 | 21 | 16 | 13 | 10 |

The premium is only paid on the gap, so it does not affect a balanced pool at all and does not enter the break-even of §2. It is a second, separately priced product: the right to hold the crowded side. Two side effects to keep in mind. It makes the dominant side more expensive than the minority side by $$\lambda_P \cdot \text{gap}/r_{\text{dom}}$$, which is the only balancing force the engine has, since the minority side receives nothing. And it compounds with interest for a one-sided book: a BTC ×4 pool at $$\text{HL}_I = 77$$ d, $$\text{HL}_P = 84$$ d charges a lone long about 1.7%/day. If that is too steep, lower $$S$$ rather than lengthening `INTEREST_HL`. Variance is the cost that never goes away.

## 4. `OPEN_RATE`

**Zero is the fair default.** Once `INTEREST_HL` is set at break-even, every holder pays per unit of time exactly what their position costs the LP over that time, and an entry fee would be a second charge on the same thing, falling hardest on whoever holds shortest. The fee has exactly one economic job: the flow that continuous pricing cannot see. A position opened minutes before a *predictable* move and closed minutes after pays almost no interest and takes the whole convexity of the move. That flow is not in the standing book, so no interest on the standing book covers it. Two facts bound the damage even at a zero fee: the winner's pay-off saturates at the asymptotic branch, so a single event can extract at most the residual $$r_C$$ however large the straddle; and the deferred part is only paid out if the Vault refills afterwards (§5).

The practical rule: an `OPEN_RATE` fee of 0 unless the asset has announced moves large enough, at the chosen $$k$$, that event-only flow would be a material share of volume; then either lower $$k$$ until the bound below is a fraction of a percent, or charge a fee near it. Two separate bounds; take the larger.

**Churn.** Interest accrues per unit time; a trader who holds for a few hours pays almost none of it. If the average holding period is $$\tau_h$$ (years), the fee contributes $$f/\tau_h$$ per unit of standing reserve per year. To run the interest below break-even for the long holders and make it back on churn:

$$
f \;\ge\; \tau_h\left(\xi\,\tfrac{1}{2}k^2\sigma^2 - \varphi\lambda_I\right) \qquad \text{(positive part only)}
$$

BTC ×4, $$\sigma = 0.55$$, $$\text{HL}_I = 180$$ d, $$\tau_h = 3$$ d gives $$f \ge 1.1\%$$; for day-trader flow ($$\tau_h = 1$$ d) 0.36%. The fee only works if the flow really is short-dated; a 30-day holder at those settings costs the LP 10% and pays 1%.

**Scheduled events.** A long plus a short of equal reserve is a pure straddle on $$p^k$$. It costs $$f$$ per unit to open, negligible interest over a few hours, and returns $$\cosh(k \ln x) - 1$$ per unit after a move. Anyone who knows an event is coming (earnings, CPI, FOMC, an index rebalance, Monday's open after a weekend) and has a view on the *size* of the move but not its direction gets paid by the LP unless

$$
f \;\ge\; \cosh(k\, j_{\text{event}}) - 1
$$

where $$j_{\text{event}}$$ is the typical absolute log move on that event. It must be a *predictable* move, one whose timing is public in advance. An unscheduled 4% day is not an event for this bound; nobody can buy the straddle the hour before it, and its cost is already in $$\sigma$$ and paid by the interest. Measure $$j_{\text{event}}$$ on the announced days: the median absolute move on earnings days for a stock, on CPI/FOMC/NFP days for an index or for crypto (the 75th percentile of all daily moves is a fair proxy without a calendar). Using the tail of all days instead inflates the bound a lot at high $$k$$.

| k \ j | 1.5% | 2% | 3% | 5% | 8% | 10% | 15% |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | 0.05% | 0.08% | 0.18% | 0.50% | 1.28% | 2.0% | 4.5% |
| 3 | 0.10% | 0.18% | 0.41% | 1.13% | 2.9% | 4.5% | 10.3% |
| 4 | 0.18% | 0.32% | 0.72% | 2.0% | 5.2% | 8.1% | 18.6% |
| 6 | 0.41% | 0.72% | 1.62% | 4.5% | 11.7% | 18.6% | 43% |
| 8 | 0.72% | 1.28% | 2.9% | 8.1% | 21% | 34% | 81% |
| 16 | 2.9% | 5.2% | 11.7% | 34% | 94% | 158% | 456% |

For crypto majors the scheduled moves (macro prints) are 1.5–3%, and at $$k \le 4$$ the bound is under 1%. For single stocks with earnings this bound is the binding constraint on leverage, not the interest: a name that moves 8% on earnings cannot carry more than ×3 with a fee traders will pay, and the pool has no pause switch. The engine's only event-time defence is the dual-basis gate (a diverged TWAP/spot pair prices each leg at its adverse bound) and the stock-token fetcher's divergence cap, which halts trades when the two sources disagree beyond it. Neither protects against a well-priced straddle bought thirty minutes before the print.

When the bound exceeds what traders will pay, do not write it into `OPEN_RATE`. Cap the fee at the tolerable level and treat the difference as a known leak: each event costs the LP about $$(\cosh(kj) - 1 - f)$$ times the reserve opened into it. If that budget is not acceptable, the answer is a lower $$k$$, not a higher fee.

Set $$\text{OPEN\_RATE} = (1 - f)\cdot 2^{128}$$, with $$f = 0$$ as the default. The fee lands on $$r_C$$ through the fee-raised LP floor; LP deposits themselves pay no fee.

## 5. Headroom

The Vault refills a pool to $$R_t = 2\max(r_A, r_B)(1 + h)$$. At target the dominant side sits at $$R / (2(1+h))$$, below the inflection by a factor $$1 + h$$, so it keeps full leverage through a price move of

$$
\Delta_{\text{full}} = (1 + h)^{1/k} - 1
$$

| k \ h | 5% | 10% | 20% | 30% | 50% | 100% |
| --- | --- | --- | --- | --- | --- | --- |
| 2 | 2.5% | 4.9% | 9.5% | 14.0% | 22.5% | 41.4% |
| 4 | 1.2% | 2.4% | 4.7% | 6.8% | 10.7% | 18.9% |
| 8 | 0.6% | 1.2% | 2.3% | 3.3% | 5.2% | 9.1% |
| 16 | 0.3% | 0.6% | 1.2% | 1.7% | 2.6% | 4.4% |

What headroom does for the LP is bound the exposure per pool per refill cycle:

$$
r_C \text{ at target} = (1 + 2h)\max(r_A, r_B) - \min(r_A, r_B)
$$

($$2rh$$ matched, $$r_A(1 + 2h)$$ one-sided), and $$r_C$$ is the most the LP can lose before the next refill. A matched book with 10% headroom puts 0.2 of the side reserve at risk; a one-sided book puts 1.2 of it at risk. Imbalance, not headroom, is what exposes the LP, and the premium (§3) is what pays for it.

The cap is a *deferral*: refilling a pool whose winner is saturated hands over $$\rho(q, R_1) - \rho(q, R_0)$$, and a pool refilled to the same headroom after every move pays the uncapped convexity over time, one refill late ([LP Economics §4](../liquidity/lp-economics.md)). So the headroom choice is a quality-of-service choice with a risk budget as a ceiling:

$$
\text{QoS floor:}\quad h \;\ge\; (1 + \sigma_d\sqrt{T_{\text{refill}}})^k - 1 \qquad\qquad
\text{risk ceiling:}\quad h \;\le\; \frac{\text{budget}}{2r} \ \text{(matched)} \ \text{ or } \ \frac{\text{budget}/r_{\text{dom}} - 1}{2} \ \text{(one-sided)}
$$

Headroom for a 1σ *daily* move (refill once a day):

| k \ σ_d | 1% | 2% | 3% | 4% | 6% |
| --- | --- | --- | --- | --- | --- |
| 2 | 2.0% | 4.0% | 6.1% | 8.2% | 12.4% |
| 4 | 4.1% | 8.2% | 12.6% | 17.0% | 26.2% |
| 8 | 8.3% | 17.2% | 26.7% | 36.9% | 59.4% |
| 16 | 17.3% | 37.3% | 60.5% | 87.3% | 154% |

For completeness, the fraction of the convexity cost the LP would still pay if the pool were **not** refilled during the move (daily reset, matched book):

| pool (σ_d) | κ/day | h = 5% | 10% | 20% | 30% | 50% | 100% |
| --- | --- | --- | --- | --- | --- | --- | --- |
| BTC ×16 (3%) | 11.5% | 0.12 | 0.22 | 0.40 | 0.54 | 0.73 | 0.95 |
| BTC ×8 (3%) | 2.9% | 0.25 | 0.45 | 0.71 | 0.85 | 0.97 | 1.00 |
| BTC ×4 (3%) | 0.72% | 0.48 | 0.75 | 0.95 | 1.00 | 1.00 | 1.00 |
| SPY ×8 (1%) | 0.32% | 0.64 | 0.89 | 1.00 | 1.00 | 1.00 | 1.00 |
| SPY ×16 (1%) | 1.28% | 0.37 | 0.62 | 0.87 | 0.97 | 1.00 | 1.00 |

Use this table to size *tail* protection. Do not use it to shorten `INTEREST_HL`: with a working refill loop the long-run cost is the 1.00 column.

## 6. Ramp

The ramp caps how fast the Vault can put fresh idle into one pool. Per rebalance the open is at most $$\text{idle}\,(1 - 2^{-\Delta t/T_{1/2}})$$, and because each cap applies to the then-current idle the bound is path-independent: over any window $$T$$, however many times the worker fires, at most $$\text{idle}\,(1 - 2^{-T/T_{1/2}})$$ of the idle can flow into one pool. Defunds are not ramped (only floored at the inflection), and the Vault clamps $$T_{1/2}$$ up to its immutable `MIN_RAMP_HL`.

Economically the ramp is the speed limit on the refill transfer of §5. A trend that keeps the winner saturated turns every refill into a payment, and the ramp is the only thing bounding how much of the Vault's idle can be paid out to one pool before governance can react. Two bounds:

$$
\text{drawdown bound (lower):}\quad T_{1/2} \;\ge\; \frac{T_{\text{trend}}}{\log_2\!\big(1/(1 - q_{\max})\big)}
\qquad\qquad
\text{fill bound (upper):}\quad T_{1/2} \;\le\; \frac{T_{\text{fill}}}{\log_2\!\big(1/(1 - g)\big)}
$$

where $$q_{\max}$$ is the fraction of idle you accept sinking into one pool over a trend of $$T_{\text{trend}}$$ days, $$g$$ the share of idle a pool's target needs, and $$T_{\text{fill}}$$ how fast a newly promoted pool should reach it.

| T_trend | q_max = 10% | 20% | 33% |
| --- | --- | --- | --- |
| 3 d | 19.7 d | 9.3 d | 5.2 d |
| 7 d | 46 d | 22 d | 12 d |
| 14 d | 92 d | 44 d | 24 d |

| T½ | idle deployable per day | days to deploy 10% / 25% / 50% of idle |
| --- | --- | --- |
| 1 d | 50% | 0.2 / 0.4 / 1.0 |
| 3 d | 21% | 0.5 / 1.2 / 3.0 |
| 7 d | 9.4% | 1.1 / 2.9 / 7 |
| 14 d | 4.8% | 2.1 / 5.8 / 14 |
| 30 d | 2.3% | 4.6 / 12.5 / 30 |

The two bounds usually conflict, and the resolution is that the fill bound only matters for a pool's first fill, while the drawdown bound matters every week. Bias toward the drawdown bound. A pool whose winner is saturated *should* be allowed to stay deleveraged for a while; that is the headroom doing its job. A half-life of one hour lets the whole idle reach a single pool within a few hours, which is the fastest possible way to convert a headroom cap into a realized loss. Something in the 7–21 day range is the defensible default for a multi-pool vault.

## 7. Measuring the inputs

The formulas take five numbers per asset.

**σ, the volatility to plan against.** Compute daily log returns of the *pool's own oracle price*, not the CEX index. For a stock-token pool that is the Pyth xStock feed, which trades around the clock on thin venues and is not the same series as the underlying's closes. Annualize with $$\sqrt{365}$$ for crypto and $$\sqrt{252}$$ for stocks. Then do not use the average: volatility clusters, and the LP has to survive the high-vol regime with the parameters chosen in the calm one. Use

$$
\sigma_{\text{ref}} = \max\big(\sigma \text{ over the last 12 months},\ \text{80th percentile of rolling-30-day } \sigma \text{ over the last 24 months}\big)
$$

For a cross pair (ARB/ETH) with both legs priced against USD, $$\sigma_{AB}^2 = \sigma_A^2 + \sigma_B^2 - 2\rho_{AB}\sigma_A\sigma_B$$. ARB at 1.10, ETH at 0.75, correlation 0.7 gives 0.79; at correlation 0.5 it is 0.97. The correlation is the fragile input, so take it from a stressed window.

**σ_d and z.** $$\sigma_d = \sigma/\sqrt{365}$$ or $$\sigma/\sqrt{252}$$. $$z = 1.5$$ by default, 2 for a pool you cannot afford to be wrong on. Stocks concentrate their variance in 6.5 hours a day and 252 days a year while interest accrues 24/365; the annual formulas handle that, but any per-day reasoning must use trading-day $$\sigma_d$$.

**Jumps (ν, j).** Crypto majors: macro prints as $$\nu \approx 12$$/yr, $$j \approx 2$$–$$3\%$$; already in $$\sigma$$, they matter only for the straddle bound. Single stocks: $$\nu = 4$$ earnings a year, $$j$$ = the median absolute earnings move over the last eight prints (mega-caps 4–6%, high-beta names 8–12%, mid-caps 10–20%). Index ETFs have no earnings; $$\nu \approx 16$$, $$j \approx 1.5$$–$$2\%$$ for FOMC/CPI/NFP. Also count the overnight gap of the underlying: an xStock feed moves at the US open to wherever the stock opens, and that gap is a jump for the pool.

**Toxicity ξ.** Random hedging flow 1.0. Momentum-driven crypto flow 1.2–1.5. Alts and anything socially driven 1.5–2. Stocks around events: model the event explicitly rather than inflating $$\xi$$.

**Holding period τ_h.** From position history if the pool has one; otherwise 1–3 days for crypto speculators, 5–20 days for stock traders. Only feeds the churn fee.

**Trader tolerance f_max.** A product decision. Perp funding on centralized venues runs around 0.01%/8h in normal markets and 0.1–0.3%/day in hot ones; power-perp products have historically cleared at daily funding well above that. A pool asking above 1–2%/day of position value will be used for very short holds only, which is fine if the opening fee is set for churn.

Stock-specific extras that are not in the formulas: market hours (the underlying does not trade for two thirds of the calendar, but the feed and the local pool do, and off-hours prices are thin); `maxAgeTrade` is a freshness window, not a market-hours switch, and a stale Pyth price is the closest thing to a pause the pool has; corporate actions are handled in the price and are not a jump unless the two sources update out of sync, which the divergence cap catches.

## 8. Presets

Illustrative starting points at $$\varphi = 0.8$$, $$z = 1.5$$, $$S = 1$$, refill once a day. The volatilities are *assumptions* typical of each name over recent years; re-measure them (§7) before deploying anything. $$k$$ is $$k_{\max}$$ rounded down to the half-step at the tolerance shown, and the other parameters follow from that $$k$$. The `f (events)` column is the straddle bound of §4, shown to inform the leverage decision; the default `OPEN_RATE` fee is 0.

{% hint style="warning" %}
The leverages in these tables were sized with a value-bleed tolerance (1%/day of position value for crypto, 0.5%/day for stocks) at $$z = 1.5$$. Section 1 and the calculator now size from a notional funding tolerance (0.2%/day and 0.1%/day by default) and a target edge, so running the §9 recipe with those defaults will not reproduce the tables exactly. It lands in the same place for the volatile names and higher for the index ETFs, where other bounds take over.
{% endhint %}

Crypto (tolerance 1%/day, $$\xi$$ as shown):

| asset | σ | σ_d | ξ | k_max | k | INTEREST_HL | daily bleed | PREMIUM_HL | f (events) | h (1σ/day) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| BTC/USD | 0.55 | 2.9% | 1.0 | 2.9 | 2.5 | 94 d | 0.73% | 147 d | 0.20% | 7.4% |
| ETH/USD | 0.75 | 3.9% | 1.0 | 2.2 | 2.0 | 79 d | 0.87% | 135 d | 0.18% | 8.0% |
| SOL/USD | 1.00 | 5.2% | 1.2 | 1.5 | 1.0 | 149 d | 0.46% | 202 d | 0.08% | 5.2% |
| ARB/USD | 1.10 | 5.8% | 1.3 | 1.3 | 1.0 | 113 d | 0.61% | 184 d | 0.13% | 5.8% |
| ARB/ETH | 0.80 | 4.2% | 1.3 | 1.8 | 1.5 | 95 d | 0.73% | 169 d | 0.18% | 6.3% |

Stocks (tolerance 0.5%/day, $$\xi = 1$$, earnings and events modelled as jumps):

| asset | σ | σ_d | ν, j | k_max | k | INTEREST_HL | daily bleed | PREMIUM_HL | f (events) | h (1σ/day) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| SPY | 0.16 | 1.0% | 16, 2% | 7.1 | 7 | 129 d | 0.54% | 181 d | 0.98% | 7.3% |
| QQQ | 0.22 | 1.4% | 16, 2.5% | 5.2 | 5 | 136 d | 0.51% | 184 d | 0.78% | 7.1% |
| AAPL / MSFT | 0.28 | 1.8% | 4, 5% | 4.1 | 4 | 136 d | 0.51% | 181 d | 2.0% | 7.2% |
| NVDA | 0.50 | 3.1% | 4, 8% | 2.3 | 2 | 172 d | 0.40% | 202 d | 1.28% | 6.4% |
| TSLA | 0.60 | 3.8% | 4, 10% | 1.9 | 1.5 | 212 d | 0.33% | 225 d | 1.13% | 5.7% |

Two things stand out. Index ETFs are the best fit for the product: low vol, no earnings, leverage in the ×5–7 range at half-percent daily interest. Single stocks are bounded by the earnings straddle before they are bounded by vol; an $$f$$ of 2% on AAPL ×4 is what it takes not to be picked off four times a year, and that is a fee stock traders will notice.

What it costs to hold a *fixed* displayed leverage instead:

| asset | ×2 | ×4 | ×8 |
| --- | --- | --- | --- |
| BTC/USD | 147 d (0.47%/d) | 37 d (1.9%/d) | 9 d (7.3%/d) |
| ETH/USD | 79 d (0.87%/d) | 20 d (3.4%/d) | 5 d (13%/d) |
| ARB/ETH | 54 d (1.3%/d) | 13 d (5.1%/d) | 3.3 d (19%/d) |
| SPY | 1581 d (0.04%/d) | 395 d (0.18%/d) | 99 d (0.70%/d) |
| AAPL | 543 d (0.13%/d) | 136 d (0.51%/d) | 34 d (2.0%/d) |
| NVDA | 172 d (0.40%/d) | 43 d (1.6%/d) | 11 d (6.3%/d) |
| TSLA | 119 d (0.58%/d) | 30 d (2.3%/d) | 7.4 d (8.9%/d) |

Ramp, all pools: 7–21 days (§6), floored by the Vault's `MIN_RAMP_HL`.

## 9. Recipe

1. Confirm the fetcher's price convention and hence whether `K = k` or `K = 2k`.
2. Measure $$\sigma_{\text{ref}}$$ on the pool's own oracle series (§7). Derive $$\sigma_d$$.
3. Decide $$f_{\max}$$, the target edge $$e$$ (or equivalently $$z$$), and $$\xi$$; $$k_{\max}$$ is the largest $$k$$ with $$\xi\tfrac{1}{2}k^2\sigma^2 + \nu(\cosh(kj) - 1) + e \le \varphi k\lambda_n$$; round down to the `K` grid.
4. $$\text{INTEREST\_HL} = \varphi \ln 2 / (\xi\tfrac{1}{2}k^2\sigma^2 + \nu(\cosh(kj) - 1) + e)$$ years, to seconds.
5. $$\text{PREMIUM\_HL} = \varphi \ln 2 / (k S \sigma)$$ years, to seconds.
6. Default $$f = 0$$. Compute the straddle bound $$\cosh(k j_{\text{event}}) - 1$$ anyway: if it is more than a fraction of a percent and the asset has announced moves, either go back to step 3 with a lower $$k$$ or charge a fee near the bound. $$\text{OPEN\_RATE} = (1 - f)\cdot 2^{128}$$.
7. $$h$$ between the QoS floor for your refill cadence and the per-pool risk budget; $$\text{headRoom} = h \cdot 65536$$.
8. $$T_{1/2}$$ from the drawdown bound; check it is above `MIN_RAMP_HL`.
9. Backtest (§10) on at least two years of the oracle series including the worst month. Ship only if the LP is flat or better over the majority of rolling weeks and the worst week is inside the risk budget.

## 10. Backtest

The analytic formulas assume lognormal returns and a matched book. Real series have fat tails and real books are lopsided, so check the parameters against the oracle history before deploying. One pass per candidate $$(k, \text{HL}_I, \text{HL}_P, f, h, T_{1/2})$$:

```
state: rA = rB = 1 (matched), R = 2·(1+h), rC = 2h, cum_lp = 0, idle = large
for each day t with log return u:
    # price move, both curves against the current R
    qA = rA_lin · e^{k·u};  qB = rB_lin · e^{−k·u}         # linear payoffs (track separately)
    rA = ρ(qA, R);  rB = ρ(qB, R)
    # funding, in reserve space
    i  = 1 − 2^(−1/HL_I_days);  rA -= rA·i;  rB -= rB·i;  release = the two decrements
    gap = |rA − rB|;  d = gap·(1 − 2^(−1/HL_P_days));  dominant side −= d;  release += d
    R  -= release·(1 − φ)                                    # protocol cut leaves; the rest is rC's
    # optional: churn — reopen a fraction 1/τ_h of each side at the new price, paying f into rC
    # refill toward target, ramp-capped
    Rt = 2·max(rA, rB)·(1+h);  dep = min(Rt − R, idle·(1 − 2^(−1/T_half_days)))  if Rt > R
    R += dep;  idle −= dep;  re-evaluate rA, rB at the new R;  cum_lp += (R − rA − rB) − rC_prev − dep
    rC_prev = R − rA − rB
report: share of rolling 7-day windows with cum_lp change ≥ 0; worst 7-day and 30-day drawdown as a fraction of idle;
        the same with the book forced one-sided (rB = 0.1·rA) to exercise the premium.
```

Run it on the worst two years you have. If the majority of weeks are not flat or positive at $$z = 1.5$$, the leverage is too high for the asset. If the weeks are fine but the worst month is outside the budget, the ramp is too fast or the headroom too large. If a one-sided book bleeds while a matched one does not, the premium half-life is too long.

## Appendix: from parameters to config

```
K            = k                (plain-price fetcher: Chainlink branch, stock tokens)
             = 2·k              (sqrt-price fetcher: Uniswap v3, Composite)
INTEREST_HL  = HL_I_days · 86400
PREMIUM_HL   = HL_P_days · 86400
OPEN_RATE    = floor((1 − f) · 2^128)
headRoom     = round(h · 65536)                 (strategy record, Q16)
rampHL       = T_half_days · 86400              (≥ Vault.MIN_RAMP_HL, else clamped up)
φ            = 1 − 1/FEE_RATE                   (0.8 at the deployed FEE_RATE = 5)
```
