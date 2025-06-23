# Funding Rate

There are two components to the Funding Rate: the Interest Rate and the Premium Rate.

Unlike conventional perpetual exchanges, where the funding rate is manually charged and updated periodically, Derion's funding rate is autonomous and continuously applied on every block of the underlying smart contract platform.

<figure><img src="../.gitbook/assets/fee.gif" alt=""><figcaption><p>Interest and Protocol Fee</p></figcaption></figure>

## Interest Rate

The Interest Rate is continuously charged from both the Long and Short sides to the LP side with a decay rate initialized by the pool, that scales with the effective leverage (or delta) of each side.

A total of 1/5 of the interest tokens paid to LP is collected as the protocol fee.

## Premium Rate

The Premium is paid by the larger to the smaller side of Long and Short, offering them the chance of negative funding rates as an incentive to balance the market.

With:

* rLong: long reserve
* rShort: short reserve
* rLP: LP reserve
* $$PR_{max}$$: the constant maximum premium rate for a single Derion pool

Long Premium Rate:

$$PR_{long} = PR_{max} \times \dfrac{rLong^2 - rShort^2}{R \times rLong}$$

Short Premium Rate:

$$PR_{short} = PR_{max} \times \dfrac{rShort^2 - rLong^2}{R \times rShort}$$
