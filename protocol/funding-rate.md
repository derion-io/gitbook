# Funding Rate

There are two components to the Funding Rate: the LP Interest Rate and the Protocol Fee Rate.

The LP Interest Rate is constantly charged from both the Long and Short sides to the LP side. This rate is configured by each pool as a fixed daily percentage and also decreases as the curve is deleveraged. That means the LP interest rate is proportional to the effective leverage of its side in a pool.

<figure><img src="../.gitbook/assets/time (1).gif" alt=""><figcaption><p>LP Interest and Protocol Fee</p></figcaption></figure>

The Protocol Fee is constantly charged from all sides of the pool, and this rate is fixed as 1/12 of the pool's LP Interest Rate.

For example, a pool initialized with a daily LP Interest Rate of 0.06%:

* Every day, 0.06% of the LONG reserve and 0.06% of the SHORT reserve will be paid to the LP as the interest rate.
* Every day, 0.005% of the total pool reserve will be paid to the Derivable Labs as the protocol fee.
