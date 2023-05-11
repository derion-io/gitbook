# Future Expiration

Derivable LONG and SHORT tokens are perpetual by nature, but future tokens can be enabled by each pool on initialization using two independent configs: MinExpiration and DiscountRate.

MinExpiration enforces all LONG and SHORT tokens to be locked for at least a specified number of seconds before they can be transferred or closed. This config is especially useful for markets with low liquidity to prevent price manipulation attacks.

DiscountRate allows each LONG and SHORT position to be opened with an expiration time and benefit from the Interest Rate discount. (0 ≤ DiscountRate ≤ 1)

$$InterestFreeTime = (Expiration - MinExpiration) \times DiscountRate$$

For example, a pool with a DiscountRate of 75% and a MinExpiration of 1 day. If a LONG (or SHORT) position is opened with an expiration of 5 days, it will be locked for 5 days, and the first 3 days will have no interest rate.
