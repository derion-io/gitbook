# Features

After recognizing the existing pain points that Derivatives DEXes are currently having, Derivable has developed multiple innovative technologies in an attempt to tackle those shortcomings.

* Power (compounding) Leverage instead of Linear Leverage: More potential upside with less downside, no liquidation risk.
* Asymptotic Power Curve as Pricing Curve (see below): Enable infinite liquidity with no liquidation. Congested liquidation events are eliminated because there simply won’t be any liquidation!
* Auto-Deleveraging (ADL): Our risk management mechanism to mitigate the effects of extreme market conditions (under-collateralization).
* Double-price system: Enforces the conversion rate to use the less beneficial result of the 2 prices (Spot & TWAP, both from Uniswap v3 Oracle).
  * Prevent price manipulation like a flashloan attack.
  * Prevent TWAP exploit.
  * Gather more fees for Liquidity Providers during volatile markets.
