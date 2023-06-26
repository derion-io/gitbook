---
description: (EXPERIMENTAL)
---

# Future Position

<mark style="color:yellow;">Experimental Feature Notice:</mark> Future Position is in an experimental state and not recommended for public use. As for now, the Derivable front-end will not show any pool with Future Discount Rate. This feature can only be used directly from the smart contract, and the user takes full responsibility for their own risk.

Users can add extra locktime to the maturity duration when opening a position, effectively converting it into a futures position. Futures position benefit from the `DiscountRate`, as the LP interest will not be charged for a portion of the locktime.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption><p>Future Position</p></figcaption></figure>

For example, suppose a pool has a Discount Rate of 75%. If a user opens a LONG or SHORT position with an additional locktime of 4 days, no interest rate will be applied during the first 3 days of the locktime.
