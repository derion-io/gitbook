# Future Position

Users can add extra locktime to the maturity duration when opening a position, effectively converting it into a futures contract. Futures contracts benefit from the `DiscountRate`, as a part of the locktime, the LP interest will not be charged.

$$InterestFreeDuration = Locktime \times DiscountRate$$

For example, consider a pool with a DiscountRate of 75%; if a LONG (or SHORT) position is opened with an additional locktime of 4 days, the first 3 days will have no interest rate.
