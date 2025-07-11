---
description: For both AMM and LOB Perpetuals
---

# Derivative Backstop Mechanism

LOB perp DEXs with backstop liquidation, like HyperLiquid, faced two "HyperLiquid 2025 exploits." One involved manipulating an illiquid asset's price to force a large, unprofitable short onto the backstop (HLP). The other saw a large ETH long strategically exploit liquidation mechanics after withdrawing collateral, again shifting millions in losses to the HLP.

A core underlying issue is that calculating the comprehensive open interest and real-time LOB depth for an entire exchange, given the volume of positions and orders, is impractical for on-chain DEXs (even with sidechains). This complexity, which scales linearly with the number of positions and orders, creates blind spots that attackers can exploit to push large, problematic positions onto the backstop. These incidents expose critical vulnerabilities in oracle dependency and the backstop's risk absorption, regardless of asset liquidity.

Derion (Derivable) curves can serve as an exchange's backstop, providing a real-time, safe approximation of open interest and a mechanism to apply market slippage corresponding to the LOB depth.

### Open Interest Approximation

Value of perpetual positions:

$$
V=C+C\cdot L \cdot \dfrac{x-x_0}{x_0}
$$

Where:

* C: entry colateral
* L: leverage
* $$x_0$$: entry price
* $$x$$: current price

<div data-full-width="false"><figure><img src="../.gitbook/assets/image (5) (1).png" alt="" width="563"><figcaption><p>Perpetual Positions</p></figcaption></figure></div>

Each position with leverage $$L\le K$$ can be positioned on the $$x^K$$ curve with a coefficient:

$$
m=\dfrac{L}K\dfrac{C}{x_0^K}
$$

The value of the position is calculated using V<sup>\*</sup> instead of V. Both have the same value and first derivative at $$x=x_0$$:

$$
V^*=C(1-{L\over K})+mx^K
$$

<figure><img src="../.gitbook/assets/image (6) (1).png" alt="" width="563"><figcaption><p>Long Open Interest</p></figcaption></figure>

It's been shown that compounding leveraged trading closely approximates power perpetuals. Given the continuous opening and closing of positions in a perpetual market, open interest can be safely approximated using power perpetual curves.

<figure><img src="../.gitbook/assets/image (6).png" alt="" width="563"><figcaption></figcaption></figure>

While deviations can occur when large positions remain untouched for extended periods (especially during price swings), the positive gamma exposure inherent in power perpetuals ensures that any such deviation from the exact open interest is always non-negative. This makes it a safer estimation for liquidity providers and market makers, favoring their position.

$$
\Delta{V}(x)=CL\left[\dfrac{1}K\left(\dfrac{x^K}{x_0^K}-1\right)-\left(\dfrac{x}{x_0}-1\right)\right]
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

While the Derion itself functions as a compounding perpetuals AMM—a concept that might be novel to many traders and market makers—its double-curve pool design offers a powerful application. It can effectively serve as a financial security layer underpinning a familiar, unique position perp DEX, significantly enhancing protection against market manipulation and exploits, especially in on-chain DEX environments where complex calculations are limited and expensive.

With Derion as the derivative backstop mechanism, attacks like those seen on HyperLiquid would be significantly mitigated, if not entirely prevented. This is because a large, exploitative position would instantly incur a massive premium due to Derion's real-time open interest approximation and slippage adjustment. Furthermore, the dynamic LOB slippage mechanism would prevent almost all collateral from being withdrawn in a manner that puts the backstop liquidator at a loss, effectively curbing the attack vectors previously exploited.
