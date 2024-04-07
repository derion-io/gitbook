# Liquidity Efficiency

In conventional perpetual DEXes with server-based ADL mechanisms, the liquidity-to-position rate often has to be highly abundant to protect the exchange from unpredictable market volatility. With smooth deleveraging curves built into the payoff function, Derion provides a much safer and more efficient liquidity provision environment.

<figure><img src="../.gitbook/assets/image (50).png" alt="" width="563"><figcaption></figcaption></figure>

Only a low rate of liquidity-to-position size is required to keep the market in its Full Leverage range. As soon as the market is about to slip out of the Full Leverage range, more liquidity can be provided without any disruption of the service or trader experience. The worst-case scenario is the market just slightly slips out of the Full Leverage range; additional liquidity is incentivized to be added while the effect on traders is barely noticeable.
