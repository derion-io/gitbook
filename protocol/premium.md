# Premium

If enabled by the pool on initialization, the Premium might be charged when users buy LONG or SHORT tokens in imbalanced pools to protect the LP from counter-party risk in extremely volatile markets.

If the pool is configured with the `PremiumRate > 0`, and we have the pool states as follows:

* rA: total reserve for LONG derivatives
* rB: total reserve for SHORT derivatives
* rC: total reserve for LP
* RiskFactor: $${rA-rB} \over rC$$

The premium for buying the LONG token is calculated as follows:

* if `RiskFactory ≤ PremiumRate`, there's no Premium Fee for buying LONG
* if `RiskFactory > PremiumRate`, the LONG token price will be $$RiskFactor \over PremiumRate$$ times higher. This rate will also decrease proportionally as the curve deleverages.

<figure><img src="../.gitbook/assets/image (5) (2).png" alt=""><figcaption><p>Premium</p></figcaption></figure>

The premium for buying the SHORT token is calculated as follows:

* if `RiskFactory ≥ -PremiumRate`, there's no Premium Fee for buying SHORT
* if `RiskFactory < -PremiumRate`, the SHORT token price will be $$|{RiskFactor \over PremiumRate}|$$ times higher. This rate will also decrease proportionally as the curve deleverages.
