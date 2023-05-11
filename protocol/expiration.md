# Future Expiration

Derivable LONG and SHORT tokens are perpetual by nature, but future tokens with expiration can be enabled by each pool on initialization using two independent configs: MinExpiration and DiscountRate.

MinExpiration enforces all LONG and SHORT tokens to be locked for at least a specified number of seconds before they can be transferred or burnt. This config is especially useful for markets with low liquidity to prevent price manipulation attacks.

DiscountRate (0 ≤ DiscountRate ≤ 1) allows each LONG and SHORT token to be minted with an expiration time and benefit from the Interest Rate discount.

$$InterestFreeTime = (Expiration - MinExpiration) \times DiscountRate$$

For example, a pool with a DiscountRate of 75% and a MinExpiration of 1 day. If a LONG (or SHORT) token amount is minted with an expiration of 5 days, it will be locked for 5 days, and the first 3 days will have no interest rate.

Token expiration has a dissolving effect, as minting a new token amount to an unexpired token balance will have the weighted average expiration of the two.
