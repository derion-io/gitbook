# Funding Rate

There are two components to the Funding Rate: the Interest Rate and the Premium Rate.

The Interest Rate is continuously charged from both the Long and Short sides to the LP side with a constant decay rate initialized by the pool.

<figure><img src="../.gitbook/assets/fee.gif" alt=""><figcaption><p>Interest and Protocol Fee</p></figcaption></figure>

The Premium Rate is charged to the larger side of Long and Short and is paid to the other two sides proportionally, offering them the chance of negative funding rates as an incentive to balance the market.

* If the pool has equal reserves for Long and Short, no premium is charged whatsoever.
* If the pool has close to 0% of Short positions, Long positions will have to pay the Maximum Premium Rate, and this premium will be split between Short and LP positions pro-rata.
* If the pool has a 40% reserve for Short, 30% for Long, and the rest for LP, Short positions will have to pay (40% - 30%) ÷ 40% = 25% of the Maximum Premium Rate, and this premium will be split between Long and LP positions pro-rata.

Unlike conventional perpetual exchanges, where the funding rate is manually charged and updated periodically, Derivable's funding rate is autonomous and continuously applied on every block of the underlying smart contract platform.

A total of 1/5 of the interest and premium tokens paid to LP is collected as the protocol fee.
