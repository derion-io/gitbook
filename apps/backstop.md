---
description: For both AMM and LOB Perpetuals
---

# Derivative Backstop Mechanism

Order-book perp DEXs with a backstop liquidator, Hyperliquid among them, took two such hits in March 2025. In the first, a trader shorted an illiquid token and pumped its spot price until the losing short was forced onto the backstop vault (HLP), which sat on an unrealized loss reported at around $13M before the market was delisted and force-settled ([CoinDesk, 26 March 2025](https://www.coindesk.com/markets/2025/03/26/hyperliquid-delists-jellyjelly-after-vault-squeezed-in-usd13m-tussle)). In the second, the holder of a $200M ETH long withdrew collateral until the position was liquidated, leaving HLP with about $4M of the loss ([Arkham Research](https://info.arkm.com/research/hyperliquid-whale-passes-4m-loss-to-hlp-vault)).

A core underlying issue is that calculating the comprehensive open interest and real-time LOB depth for an entire exchange, given the volume of positions and orders, is impractical for on-chain DEXs (even with sidechains). This complexity, which scales linearly with the number of positions and orders, creates blind spots that attackers can exploit to push large, problematic positions onto the backstop. These incidents expose critical vulnerabilities in oracle dependency and the backstop's risk absorption, regardless of asset liquidity.

Derion curves can serve as an exchange's backstop, providing a real-time, safe approximation of open interest and a mechanism to apply market slippage corresponding to the LOB depth.

### Open Interest Approximation

Value of perpetual positions:

$$
V=C+C\cdot L \cdot \dfrac{x-x_0}{x_0}
$$

Where:

* C: entry collateral
* L: leverage
* $$x_0$$: entry price
* $$x$$: current price

<div data-full-width="false"><figure><img src="../.gitbook/assets/image (5) (1).png" alt="" width="563"><figcaption><p>Perpetual Positions</p></figcaption></figure></div>

Each position with leverage $$L\le k$$ can be positioned on the $$x^k$$ curve with a coefficient:

$$
m=\dfrac{L}k\dfrac{C}{x_0^k}
$$

The value of the position is calculated using V<sup>\*</sup> instead of V. Both have the same value and first derivative at $$x=x_0$$:

$$
V^*=C(1-{L\over k})+mx^k
$$

<figure><img src="../.gitbook/assets/image (6) (1).png" alt="" width="563"><figcaption><p>Long Open Interest</p></figcaption></figure>

It's been shown that compounding leveraged trading closely approximates power perpetuals. Given the continuous opening and closing of positions in a perpetual market, open interest can be safely approximated using power perpetual curves.

<figure><img src="../.gitbook/assets/image (6).png" alt="" width="563"><figcaption></figcaption></figure>

While deviations can occur when large positions remain untouched for extended periods (especially during price swings), the positive gamma exposure inherent in power perpetuals ensures that any such deviation from the exact open interest is always non-negative. This makes it a safer estimation for liquidity providers and market makers, favoring their position.

$$
\Delta{V}(x)=CL\left[\dfrac{1}k\left(\dfrac{x^k}{x_0^k}-1\right)-\left(\dfrac{x}{x_0}-1\right)\right]
$$

Along with market traders' natural position actions (open and close), each position can be trustlessly "repositioned" onto the curve by anyone while retaining its full value and properties. This allows the market maker to control the deviation from the exact open interest as precisely as desired, limited only by the on-chain transaction frequency and cost.

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt="" width="563"><figcaption><p>Position Repositioning</p></figcaption></figure>

With a real-time, safe approximation of open interest, perp DEXs can constantly adjust their reflective premium rate at every moment, effectively eliminating any premium rate gaps ripe for exploitation.

### LOB Slippage Feedback

In Limit Order Book (LOB) markets, closing a large position typically incurs significant slippage, and neglecting this factor can lead to substantial losses for market makers. The "deleverage curve" can provide feedback on the LOB's depth when one side of the market becomes dominant.

<figure><img src="../.gitbook/assets/image (3) (1).png" alt="" width="563"><figcaption></figcaption></figure>

The LOB slippage can be adjusted in real-time by dynamically adding or removing pool liquidity to accurately reflect the market's true LOB depth.

<figure><img src="../.gitbook/assets/image (4) (1).png" alt="" width="345"><figcaption></figcaption></figure>

### Summary

Derion itself is a compounding perpetuals AMM, a concept that may be new to many traders and market makers, but its two-curve pool design has a second use. It can serve as a security layer under a conventional per-position perp DEX, adding protection against market manipulation and exploits, especially on-chain, where complex calculations are limited and expensive.

With Derion as the derivative backstop mechanism, attacks like those seen on Hyperliquid would be significantly mitigated, if not entirely prevented. This is because a large, exploitative position would instantly incur a massive premium due to Derion's real-time open interest approximation and slippage adjustment. Furthermore, the dynamic LOB slippage mechanism would prevent almost all collateral from being withdrawn in a manner that puts the backstop liquidator at a loss, effectively curbing the attack vectors previously exploited.
