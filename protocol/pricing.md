---
description: Asymptotic Power Curves
---

# Pay-off Curve

The Long pay-off value:

$$
\Phi(k,x)=\begin{cases} \begin{align*} \alpha x^k\quad &\text{if }x\le\sqrt[k]\frac{R}{2\alpha} \\R-\frac{R^2}{4\alpha x^k}\quad&\text{overwise} \end{align*} \end{cases}
$$

The Short pay-off value:

$$
\Psi(k,x)=\begin{cases} \begin{align*} \beta x^{-k}\quad &\text{if }x\ge\sqrt[k]\frac{2\beta}{R} \\R-\frac{R^2 x^k}{4\beta}\quad&\text{overwise} \end{align*} \end{cases}
$$

The Liquidity pay-off value:

$$
\Omega(k,x)=R-\Phi(k,x)-\Psi(k,x)
$$

<figure><img src="../.gitbook/assets/image (13).png" alt="" width="563"><figcaption></figcaption></figure>

Asymptotic Power Curves is our innovative Pay-off Curve design that makes the vision of Derion feasible in the first place. Thanks to the Asymptotic Power Curves, Derion is able to achieve:

* NO liquidation at all;
* Everlasting perpetual future market for all market circumstances;
* Infinite liquidity, even without liquidity providers;
* Liquidity elasticity: Impermanent loss and gain, with asymptotic behavior at infinity;
* Leverage elasticity: Leverage is automatically and continuously capped upon market state transitions and price volatility.
* Continuous interest rate: to compensate LPs and help them reduce the risk of impermanent loss
