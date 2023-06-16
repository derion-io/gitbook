# Future Position

Users can add extra locktime to the maturity duration when opening a position, effectively converting it into a futures position. Futures position benefit from the `DiscountRate`, as the LP interest will not be charged for a portion of the locktime.

$$InterestFreeDuration = Locktime \times DiscountRate$$

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

For example, consider a pool with a `DiscountRate` of 75%; if a LONG (or SHORT) position is opened with an additional locktime of 4 days, the first 3 days will have no interest rate.
