# Maturity

All Derion positions have a maturity period. Before the final maturity date is reached (the maturity duration expires), the position's payoff is lower than its face value. The position’s payoff will be gradually increased towards the face value until the maturity duration ends, at which the position’s payoff = face value.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

The maturity duration can be configured by the pool, creating a soft-lock effect for newly opened positions to protect markets with low-cap, high-volatility indexes from price manipulation attacks. Maturity achieves the protection ability by penalizing users who want to open and close positions in an extremely short period of time in an attempt to manipulate prices.

Once the maturity time has elapsed, the position becomes fully matured and fully fungible. However, until that time, the position is only partly fungible and subject to the following rules:

* A maturing position can be divided into smaller positions with the same maturity time.
* A maturing position cannot be transferred or merged into a more mature position.
* Merging two positions will result in a position with a later maturity time.

Closing a maturing position will result in a payoff of zero or less than the position's face value, which is calculated using the following configurations:

* `MATURITY` the full maturity duration, after which the position payoff will be its full face value.
* `MATURITY_RATE` the maximum payoff rate for premature position closing.
* `MATURITY_VEST` the vesting period from the opening time, where the payoff is linearly increased from zero to `MATURITY_RATE` of the face value.

The Maturity design offers derivative users greater financial flexibility than the Locktime design. With Maturity, users can sacrifice a portion of their position value in exchange for capital freedom while maintaining the system's resistance to manipulation.
