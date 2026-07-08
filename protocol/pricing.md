---
description: The asymptotic power curve
---

# Pay-off Curve

Everything in a Derion pool is priced by a single **asymptotic deleverage curve**. For a claim of ideal value $$q$$ against an engine reserve $$R$$:

$$
\rho(q,R)=\begin{cases} \begin{align*} q\quad &\text{if }q\le\frac{R}{2} \\ R-\frac{R^2}{4q}\quad&\text{if }q\ge\frac{R}{2} \end{align*} \end{cases}
$$

Below the inflection point $$q = R/2$$ the claim is paid in full — the **power branch**, where leverage compounds at the full power $$k$$. Above it, the pay-off bends onto the **asymptotic branch** and approaches $$R$$ without ever reaching it. The two branches meet smoothly at the inflection point, with slope 1 on both sides.

With the normalized index price $$x = p\,/\,\text{mark}$$, the Long side's reserve is

$$
r_A=\rho(\alpha x^k,\,R)
$$

and the Short side is simply the remainder:

$$
r_B=R-r_A
$$

The complement is not a bookkeeping shortcut — it is itself an asymptotic short pay-off. Algebraically, $$R-\rho(\alpha x^k,R)=\rho(\beta x^{-k},R)$$ with the implied short coefficient $$\beta = R^2/4\alpha$$, so the two sides are exact mirror images around the inflection point: whenever one side is on its power branch, the other is on its asymptotic branch.

Three properties of $$\rho$$ carry the whole protocol:

* **No liquidation, ever.** $$\rho \to R$$ asymptotically, so the winning side can never claim the entire reserve and the losing side only approaches zero. $$0 \le r_A, r_B < R$$ holds at every price, unconditionally — even in a pool untouched for a year.
* **Leverage elasticity.** As a side saturates, its effective leverage compresses continuously toward zero instead of hitting a margin call. Deleveraging replaces liquidation.
* **Scale-free.** $$\rho(f q,\,f R) = f\,\rho(q,R)$$ — the identity that lets [funding](funding-rate.md) be applied as a single multiplication on $$R$$.

### What a trader experiences

A Long ×4 opened with 1.0 reserve: if the index rises 10%, it closes at ≈ 1.4641 (+46.41%); if the index falls 10%, ≈ 0.6561 (−34.39%); if the index crashes toward zero, a small residual remains — never zero, never liquidated. A Short is the mirror image. Compounding cuts both ways: larger gains on the right side, softened losses on the wrong side, no margin call in either direction.

### Why a power curve, not a linear pay-off

"Constant leverage" and "linear pay-off" sound like synonyms; they are opposites. Leverage as a trader experiences it is elasticity — $$d\ln V / d\ln p$$. Demanding that it stay constant at $$k$$ forces $$V \propto p^k$$: a one-line differential equation with a unique solution, and that solution is a power curve.

A pay-off linear *in price*, $$V = V_0(1 + \lambda(p/p_0 - 1))$$, only has leverage $$\lambda$$ at its entry price. It decays toward 1× as the position wins, blows up toward infinity as it loses, and crosses zero at a finite price — and that zero crossing *is* a liquidation price. Every system built on linear pay-offs pays one of two taxes:

* **Liquidation** (margin perps): the pay-off crosses zero, so solvency depends on keepers closing positions in time — bad debt, insurance funds, auto-deleveraging, oracle-latency risk. Solvency becomes an operational process instead of a property of the math.
* **Re-anchoring** (leveraged-ETF style tokens): reset the pay-off every epoch — path-dependent volatility drift, and value stops being a function of the current price alone.

Both also destroy fungibility. $$p^k$$ is a pure function of the current price and composes multiplicatively, so every holder of a side owns shares of one common pay-off — no entry price, no per-account margin, no per-position funding bookkeeping. A linear pay-off cannot be stateless: its value depends on where it was anchored.

The two branches of $$\rho$$ then split the work cleanly. The power branch is the product — constant compounding leverage. The asymptotic branch is not about leverage at all — it is the solvency clamp that replaces liquidating the counterparty. The cost of the curve is equally explicit: its convexity has to be paid for — that is what [funding](funding-rate.md) is, the same reason power perps pay theta — and a saturated side's leverage dilutes until [depth](../vault/depth-provision.md) arrives. What that buys is unconditional full collateralization and stateless, fungible positions — a combination no linear pay-off can produce at any level of added machinery.
