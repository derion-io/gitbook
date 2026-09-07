---
description: The asymptotic power curve
---

# Pay-off Curve

A perpetual on an exchange pays a straight line. At ×4, a 10% move in the index is a 40% change in your margin either way, and a 25% move against you is a liquidation. A Derion Long ×4 is worth the index price to the fourth power, and a Short ×4 the inverse fourth power. The same moves come out differently:

| Index move | Linear ×4 | Derion Long ×4 |
| --- | --- | --- |
| +25% | +100% | +144% |
| +10% | +40% | +46% |
| −10% | −40% | −34% |
| −25% | liquidated | −68% |
| −50% | liquidated | −94% |
| toward zero | liquidated | a small residual, never zero |

Gains accelerate on the right side, losses decelerate on the wrong side, and there is no price at which the position is taken from you. A Short is the mirror image. What you pay for the shape is [funding](funding-rate.md).

<figure><img src="../.gitbook/assets/image (5).png" alt="" width="563"><figcaption></figcaption></figure>

### The curve

Everything in a Derion pool is priced by a single **asymptotic deleverage curve**. For a claim of ideal value $$q$$ against an engine reserve $$R$$:

$$
\rho(q,R)=\begin{cases} \begin{align*} q\quad &\text{if }q\le\frac{R}{2} \\ R-\frac{R^2}{4q}\quad&\text{if }q\ge\frac{R}{2} \end{align*} \end{cases}
$$

Below the inflection point $$q = R/2$$ the claim is paid in full. This is the **power branch**, where leverage compounds at the full power $$k$$ and the table above holds exactly. Above it, the pay-off bends onto the **asymptotic branch** and approaches $$R$$ without ever reaching it. The two branches meet smoothly at the inflection point, with slope 1 on both sides.

### Two curves

With the normalized index price $$x = p\,/\,\text{mark}$$, each side has its own coefficient and its own curve:

$$
r_A=\rho(\alpha x^k,\,R) \qquad\qquad r_B=\rho(\beta x^{-k},\,R)
$$

The Long is a true $$p^k$$ pay-off below its inflection and the Short a true $$p^{-k}$$ pay-off below its own. Constant compounding leverage holds on both sides at any imbalance, not only near balance, and neither curve is derived from the other. What the two curves leave of the reserve,

$$
r_C = R - r_A - r_B
$$

is the [LP class](engine.md). It has no coefficient and no curve of its own; its pay-off is whatever the two power curves do not claim.

An earlier engine used a single coefficient and defined the Short as the complement $$R - r_A$$. That is the boundary case $$\alpha\beta = (R/2)^2$$ of this one, where $$r_B \equiv R - r_A$$ and $$r_C \equiv 0$$; below its inflection that Short was linear in $$p^k$$ rather than a $$p^{-k}$$ pay-off, so "a Short is the mirror of a Long" held only near balance. With two coefficients it holds everywhere.

### Three properties

Three properties of $$\rho$$ carry the whole protocol:

* **No liquidation, ever.** $$\rho \to R$$ asymptotically, so the winning side can never claim the entire reserve and the losing side only approaches zero. $$0 \le r_A, r_B < R$$ holds at every price, unconditionally, even in a pool untouched for a year.
* **Leverage elasticity.** As a side saturates, its effective leverage compresses continuously toward zero instead of hitting a margin call. Deleveraging replaces liquidation.
* **Scale-free.** $$\rho(f q,\,f R) = f\,\rho(q,R)$$: the shape of the curve does not depend on the pool's size, so small and large pools behave identically.

### Deleveraging, from the trader's seat

The asymptotic branch is what the interface calls deleverage risk. While your side's claim is below half the pool's reserve, you are on the power branch and every 1% of index is $$k$$% of position. Once the claim would exceed that, the pool cannot pay more than its reserve, so further gains flatten toward the cap. Nothing is closed and nothing is taken away: the position is still worth more after every favorable move, only by less than the power curve would pay, and what it cannot yet claim is paid out as more reserve arrives. The [LP class](engine.md) is that reserve. A deeper LP class puts the inflection further out on both sides at once, which is what [Depth Provision](../liquidity/depth-provision.md) manages, and a position sized small relative to the pool stays on the power branch through larger moves.

For a liquidity provider the same branch is the reason there is no bad debt to socialize. A winner is paid out of the reserve and only ever a fraction of it, so the pool's promise is always covered by what it holds.

### Why a power curve, not a linear pay-off

"Constant leverage" and "linear pay-off" sound like synonyms; they are opposites. Leverage as a trader experiences it is elasticity, $$d\ln V / d\ln p$$. Demanding that it stay constant at $$k$$ forces $$V \propto p^k$$: a one-line differential equation with a unique solution, and that solution is a power curve.

A pay-off linear *in price*, $$V = V_0(1 + \lambda(p/p_0 - 1))$$, only has leverage $$\lambda$$ at its entry price. It decays toward 1× as the position wins, blows up toward infinity as it loses, and crosses zero at a finite price. That zero crossing *is* a liquidation price. Every system built on linear pay-offs pays one of two taxes:

* **Liquidation** (margin perps): the pay-off crosses zero, so solvency depends on keepers closing positions in time. Bad debt, insurance funds, auto-deleveraging, oracle-latency risk. Solvency becomes an operational process instead of a property of the math.
* **Re-anchoring** (leveraged-ETF style tokens): reset the pay-off every epoch. Path-dependent volatility drift, and value stops being a function of the current price alone.

Both also destroy fungibility. $$p^k$$ is a pure function of the current price and composes multiplicatively, so every holder of a side owns shares of one common pay-off: no entry price, no per-account margin, no per-position funding bookkeeping. A linear pay-off cannot be stateless; its value depends on where it was anchored.

The two branches of $$\rho$$ then split the work cleanly. The power branch is the product, constant compounding leverage. The asymptotic branch is not about leverage at all; it is the solvency clamp that replaces liquidating the counterparty. The cost of the curve is equally explicit: its convexity has to be paid for, which is what [funding](funding-rate.md) is (the same reason power perps pay theta), and a saturated side's leverage dilutes until [depth](../liquidity/depth-provision.md) arrives. What that buys is unconditional full collateralization and stateless, fungible positions, a combination no linear pay-off can produce at any level of added machinery.

{% hint style="info" %}
Throughout these pages $$k$$ is the pay-off power, the leverage a trader sees. How it maps onto the pool's `K` config depends on the oracle's price convention: `K = 2k` for the built-in Uniswap v3 fetcher, which reports square-root prices, and `K = k` for plain-price fetchers such as Chainlink or the [stock-token fetcher](oracle/stock-tokens.md). See [Pool Creation](../guide/pool-creation.md).
{% endhint %}
