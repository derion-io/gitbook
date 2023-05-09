# Premium

The Premium might be charged when users buy LONG or SHORT tokens in imbalanced pools. The Premium Rate can be initialized by each pool to protect the LP from counter-party risk in extremely volatile markets.

If the pool is configured with the `PremiumRate` (e.g. 0.5 or 50%), and we have the pool states as follows:

* rA: total reserve for LONG derivatives
* rB: total reserve for SHORT derivatives
* rC: total reserve for LP
* LongRiskFactor: $${rA-rB} \over rC$$

The premium for buying the LONG token is calculated as follows:

* if `LongRiskFactory ≤ PremiumRate`, there's no Premium Fee for buying LONG
* if `LongRiskFactory > PremiumRate`, the LONG token price will be $$LongRiskFactor \over PremiumRate$$ times higher.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption><p>Premium</p></figcaption></figure>

TODO: formula for Premium on deleveraged side of the curve.

The same calculation can be used for buying the SHORT token.
