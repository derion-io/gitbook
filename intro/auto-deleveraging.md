# Auto-Deleveraging

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption><p>Auto Deleveraging</p></figcaption></figure>

Thanks to Opyn Squeeth, we can see the Power Perpetual derivative $$ETH^2$$ in action; however, Squeeth is constructed as a mere synthetic asset with an ultra-volatile index. (Think of MakerDAO with DAI token pegged to $$ETH^2$$ instead of USD). The synthetic nature carries all its risks and limitations that prevent it from being open to more derivatives like higher leverages, short positions, and low-cap, high-volatility indexes.

One of the most important differences between a synthetic asset and a derivative is the Auto-Deleveraging (ADL) mechanism. Unlike synthetics, derivatives are designed for high-leverage, high-volatility indexes, so a risk-managing mechanism like ADL is essential for any viable derivatives system.

In synthetic protocols, it's generally unacceptable for a synth to lose its peg; however, for a derivative to survive any extreme market conditions, losing leverage (or deleveraging) must be a part of the contract between derivatives traders and providers.

Deleveraging is not "losing"; it is "capped profit". Essentially, in a derivative market, your profit is capped by your counterparty loss.
