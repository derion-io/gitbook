---
description: Break-even analysis of the pinning strategy vs CEX perpetual funding
---

# LP Economics

This page works out what the [depth-provision strategy](depth-provision.md) costs and earns: the **break-even funding rate** for a Vault that keeps a pool's dominant side pinned at the inflection point, and how that compares with real-world CEX perpetual funding levels. BTC with $$k = 2$$ is the running example.

## 1. The position being analyzed

Under the pinning strategy, the dominant trader side sits at the inflection point, $$r_A \approx R/2$$, so the complement $$r_B \approx R/2$$ is held almost entirely by the Vault. The Vault's capital at risk per pool is therefore roughly half the engine reserve, standing as the counterparty to the traders' net position.

Its income is the pool's [funding](../protocol/funding-rate.md):

* **Premium** — with traders crowded on one side, the trader-held gap is large ($$t_A - t_B \approx R/2$$), so the premium decay is charged against the dominant trader side and flushed to the Vault. Near the pin, this is the steady income stream.
* **Interest** — the decay of $$R$$ itself. Its incidence follows the curve: a side pays in proportion to how saturated it is, so interest collected *from traders* is small right at the pin and grows as the dominant side pushes past its inflection between rebalances. (The decay attributed to the Vault's own position flows back to itself.)

Write $$r_f$$ for the total net funding income per unit of Vault capital per year, from both components combined.

## 2. The cost: negative gamma

The Vault is short convexity. At the inflection point, the curve's second derivative gives the counterparty's gamma:

$$
\Gamma_{\text{LP}} \approx -\frac{k(k-1)\,R}{2x^2}
$$

For a price following a geometric Brownian motion with volatility σ, the impermanent-loss bleed is the usual half-gamma-sigma-squared:

$$
\frac{dIL}{dt} = \tfrac12\,|\Gamma_{\text{LP}}|\,\sigma^2 x^2 = \frac{k(k-1)\,R\,\sigma^2}{4}
$$

Per unit of Vault capital ($$\approx R/2$$):

$$
\boxed{\;\text{IL rate} = \frac{k(k-1)\,\sigma^2}{2}\;}
$$

## 3. Break-even

Setting funding income against the IL bleed:

$$
\boxed{\;r_f^{\text{break-even}} = \frac{k(k-1)\,\sigma^2}{2}\;}
$$

For $$k = 2$$ this collapses to something memorable:

$$
r_f^{\text{break-even}} = \sigma^2
$$

The break-even funding rate equals the underlying's **variance**. This is no coincidence — it mirrors option theta. A power-2 perpetual's gamma cost is proportional to variance, exactly as an ATM option's theta is $$\tfrac12\Gamma\sigma^2 S^2$$; the Vault is effectively writing a power perpetual, and σ² is its theoretical fair premium. (Opyn's Squeeth, also a power-2 perp, arrives at the same σ²-per-year funding.)

With the protocol's cut of the funding flush (currently 1/5), the gross pool funding must run about 1.25× the net break-even.

### Numbers by volatility

Using $$r_f = k(k-1)\sigma^2/2$$:

| k | σ = 40% | σ = 50% | σ = 60% | σ = 70% | σ = 80% |
| - | ------- | ------- | ------- | ------- | ------- |
| **2** | **16%** | **25%** | **36%** | **49%** | **64%** |
| 4 | 96% | 150% | 216% | 294% | 384% |
| 8 | 448% | 700% | 1008% | 1372% | 1792% |

The $$k(k-1)$$ scaling means high-leverage pools need dramatically higher funding — only short-term positions are economical for traders there, which is the intended trade-off.

## 4. Comparison with Binance perpetual funding

Binance charges BTC perpetual funding every 8 hours, with a baseline of 0.01% per 8h (~10.95% annualized) when perp tracks spot. In practice:

| Market regime | Typical 8h rate | Annualized |
| ------------- | --------------- | ---------- |
| Baseline / neutral | 0.0100% | ~10.95% |
| Mild bull | 0.01–0.03% | ~11–33% |
| Strong bull (2024 BTC run) | 0.03–0.06% | ~33–66% |
| Extreme euphoria | 0.05–0.15% | ~55–165% |
| Bear / capitulation | −0.01 to 0.00% | ~−11 to 0% |

BTC aggregate funding was overwhelmingly positive in 2024 (only 26 days negative); OI-weighted funding has been positive well over 85% of the time across the past two years, with a long-run average around 11–22% annualized and peaks above 100%.

| Scenario | BTC σ | Derion needs | Binance pays | Verdict |
| -------- | ----- | ------------ | ------------ | ------- |
| Low vol | 40% | 16% | ~11–15% | tight / underwater |
| Normal vol | 50% | 25% | ~15–25% | marginal |
| Elevated vol + bull | 60% | 36% | ~25–60% | often covered |
| High vol + euphoria | 70%+ | 49%+ | ~50–100%+ | typically covered |

The crucial nuance is that **high volatility and high funding occur together**: bull markets produce both the σ that raises the Vault's cost and the funding demand that raises its income. During euphoria, CEX funding exceeds σ²; in quiet regimes, base funding falls short of it. Structurally, the position is short volatility — which is precisely what a liquidity provider is.

What the trader buys for that funding is real: at the same 2× exposure, a CEX perp carries a liquidation price (~50% adverse move) and zero convexity, while the Derion position has positive gamma and no liquidation at any price. Traders pay for convexity; the Vault's σ² is the fair price of writing it.

## 5. Practical considerations

* **Rebalancing is discrete.** Tracking error between rebalances raises the effective requirement above the theoretical bound — treat the break-even as a floor.
* **Two income streams hedge each other's regime.** Premium pays steadily while the pool is pinned and imbalanced; interest picks up as the dominant side saturates between rebalances.
* **Protocol cut and gas** both come out of gross income before depositors see it.
* **Rate configuration is per pool** — see [Pool Creation](../guide/pool-creation.md) for suggested half-lives by asset volatility.
