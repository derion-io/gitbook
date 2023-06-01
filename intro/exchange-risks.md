# Exchange Risks

Derivative exchanges have to deal with many types of risks, and each type of risk further complicates the pricing model.

* Under-collateralization: when total user position values enlarge past the reserve liquidity of the exchange.
* Mass liquidation congression: when the market is volatile, a large number of high-leverage positions will need to be liquidated as soon as possible.
* Imbalance exposure: when the total position size of Long is much larger than Short (or vice versa), the derivative provider (exchange) is forced to take the unfavorable side of the market.
* Price Manipulation: when the index prices is manipulated to unfairly extract value from the exchange liquidity reserves.

Challenges in addressing these risks:

* The Auto-Deleveraging (ADL) mechanism is designed to counter the under-collateralization risk, which introduces much more complexity to the application design.
* Liquidating many user positions requires high throughput, high peak capacity, and predictable systems. This solution is extremely expensive in blockchain as the technology is not (yet) designed for high-performance applications like this.
* Imbalance exposure is controlled by adding the premium fee to the funding rate, which further complicates the funding rate calculation.
* Price manipulation risks require exchanges to fetch and compose index prices from multiple sources. The index prices are also restricted to only high-cap, low-volatility markets.

