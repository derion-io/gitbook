# Premium

If configured by the pool, a premium may be charged when users open LONG or SHORT positions in imbalanced pools to protect the LP from counter-party risk in extremely volatile markets.

Consider that the pool is configured with a PremiumRate > 0 and that we have the pool states as follows:

* rA: total reserve for LONG positions
* rB: total reserve for SHORT positions
* rC: total reserve for LP
* RiskFactor: $${rA-rB} \over rC$$

The premium for opening a LONG position is calculated as follows:

* if `RiskFactory ≤ PremiumRate`, there's no Premium Fee for opening LONG
* if `RiskFactory > PremiumRate`, the LONG position price will be $$RiskFactor \over PremiumRate$$ times higher. This rate will also decrease proportionally as the curve deleverages.

<figure><img src="../.gitbook/assets/image (5) (2).png" alt=""><figcaption><p>Premium</p></figcaption></figure>

The premium for opening the SHORT position is calculated as follows:

* if `RiskFactory ≥ -PremiumRate`, there's no Premium Fee for opening SHORT
* if `RiskFactory < -PremiumRate`, the SHORT position price will be $$|{RiskFactor \over PremiumRate}|$$ times higher. This rate will also decrease proportionally as the curve deleverages.
