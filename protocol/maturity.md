# Maturity

Derivable positions can have an optional maturity date, before which the position cannot be closed or transferred.

If configured by the pool, all positions must have a minimum maturity duration of no less than `MinMaturity`. This feature is designed to protect markets with low-cap, high-volatility indexes from price manipulation attacks.

Users can add additional maturity duration on top of `MinMaturity` when opening a Derivative position in order to benefit from the maturity `DiscountRate` (if configured by the pool).

DiscountRate (0 ≤ DiscountRate ≤ 1) allows each LONG and SHORT token to be minted with an expiration time and benefit from the Interest Rate discount. No interest is charged for the first `InterestFeeTime` seconds after the position is opened.&#x20;

$$InterestFreeTime = (Maturity - MinMaturity) \times DiscountRate$$

For example, consider a pool with a DiscountRate of 75% and a MinMaturity of 1 day. If a LONG (or SHORT) position is opened with a maturity of 5 days, it will be locked for 5 days, and the first 3 days will have no interest rate.

Position maturity has a diluting effect, as adding more value to an unmatured position will result in the weighted average maturity of the two positions.
