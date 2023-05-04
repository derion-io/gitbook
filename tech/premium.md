# Premium

The Premium is charged in LONG and SHORT buying transactions to protect the LP from extremely volatile markets. The Premium Rate is optionally configured by each pool, and it defines the price rate increase of the derivative token when the market is one-sided.

If the pool is configured with the `PremiumRate` (e.g. 0.6 or 60%), and we have the pool states as forllows:

* rA: total reserve for LONG derivatives
* rB: total reserve for SHORT derivatives
* rC: total reserve for LP
* LongRiskFactor: $${rA-rB} \over rC$$

The premium for buying Long is calculated as follows:

* if `LongRiskFactory ≤ PremiumRate`, there's no Premium Fee for buying LONG
* if `LongRiskFactory > PremiumRate`, the LONG token price will be $$LongRiskFactor \over PremiumRate$$ times higher.

TODO: formula for Premium on deleveraged side of the curve.
