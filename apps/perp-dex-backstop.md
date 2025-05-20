---
description: For both AMM and LOB Perpetual DEXs
---

# Perp DEX Backstop

LOB perp DEXs with backstop liquidation, like HyperLiquid, faced two "HyperLiquid 2025 exploits." One involved **manipulating an illiquid asset's price** to force a large, unprofitable short onto the backstop (HLP). The other saw a large ETH long **strategically exploit liquidation mechanics** after withdrawing collateral, again shifting millions in losses to the HLP.

A core underlying issue is that **calculating the comprehensive open interest and real-time LOB depth for an entire exchange, given the volume of positions and orders, is impractical for on-chain DEXs (even with sidechains)**. This complexity, which scales linearly with the number of positions and orders, creates blind spots that attackers can exploit to push large, problematic positions onto the backstop. These incidents expose critical vulnerabilities in oracle dependency and the backstop's risk absorption, regardless of asset liquidity.

**Derion (Derivable) curves** can serve as an exchange's backstop, providing a **real-time, safe approximation of open interest** and a mechanism to **apply market slippage corresponding to the LOB depth**.

Value of perpetual positions:

$$
V=C+C\cdot L \cdot \dfrac{\Delta{p}}{p_0}
$$

Where:

* C: entry colateral
* L: leverage
* $$p_0$$: entry price
* $$\Delta p$$: price deviation from $$p_0$$

<div data-full-width="false"><figure><img src="../.gitbook/assets/image.png" alt="" width="563"><figcaption><p>Open Interest</p></figcaption></figure></div>

Long open interest is the sum of all Position value:

$$OI=\displaystyle\sum_{i=0}^n V_i$$

To have a&#x20;

<figure><img src="../.gitbook/assets/image (1).png" alt="" width="563"><figcaption></figcaption></figure>
