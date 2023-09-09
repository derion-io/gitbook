# Auto-Deleveraging

Thanks to Opyn Squeeth, we can see the Power Perpetuals derivative ETH2 in action; however, Squeeth is constructed as a mere synthetic asset with an ultra-volatile index. (Think of MakerDAO with DAI token pegged to ETH² instead of USD). The synthetic nature carries all its risks and limitations that prevent it from being open to more derivatives like higher leverages, short positions, and low-cap, high-volatility indexes.

In synthetic protocols, it's generally unacceptable for a synth to lose its peg; however, for a derivative to survive any extreme market conditions, losing leverage (or deleveraging) must be a part of the contract between derivatives traders and providers.

<figure><img src="../.gitbook/assets/image (16).png" alt="" width="563"><figcaption><p>Auto-Deleverage</p></figcaption></figure>

Let’s use LUNA as an example. The downfall of Terra last year resulted in the death spiral run of LUNA’s price - from $80 to nearly $0. At that time, anyone could short LUNA and gain massive profits. However, the situation was obviously too extreme for users to try to long LUNA. The question here is, how could short traders earn profits when no other was losing?

The answer is that the exchange facilitating LUNA trading had to pay the profits, meaning the exchange would be in bad debt. In order to avoid this, ADL is activated, reducing the leverage of traders in profit and, if needed, closing their positions completely. ADL is imperative because it matches the liquidity of both sides in derivatives trading so that traders can effectively trade against each other without the exchange taking part in the process.

ADL is an incredibly powerful feature, and in no way should it be considered a menace for traders. Deleveraging is not "losing"; it is "capped profit". In a derivatives market, your profit is capped by your counterparty loss. Currently, most leading CEXes (BitMEX, Binance, Bybit, etc.) implement ADL in their system to protect themselves and their traders. ADL is not something completely new or innovative, but unknown to the majority of traders as they are rarely influenced by this event.
