# Maturity

All Derivable positions have a maturity period during which the position's payoff is lower than its actual value. The maturity duration can be configured by the pool, which creates a soft-lock effect for newly opened positions and protects markets with low-cap, high-volatility indexes from price manipulation attacks.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption><p>Position Maturity</p></figcaption></figure>

Once the maturity time has elapsed, the position becomes fully matured and fully fungible. However, until that time, the position is only partly fungible and subject to the following rules:

* A maturing position can be divided into smaller positions with the same maturity time.
* A maturing position cannot be transferred or merged into a more matured position.
* Merging two positions will result in a position with a later maturity time.

Closing a maturing position will result in a payoff of zero or less than the position's value, which is calculated as follows:

$$
v\times θ\times(1-2^{ε({{T-t}\over M}-1)})
$$



Where:

* $$v$$ = the position's value when fully matured
* $$T$$ = the maturity time of the position
* $$t$$ = the current `block.timestamp`

Configurations:

* $$M$$= the maturity duration configuration of the pool
* θ = the coefficient configuration
* ε = the exponential configuration

The Maturity Payoff design offers greater financial flexibility for derivative users compared to the Locktime design. With Maturity Payoff, users have the option to sacrifice a portion of their position value in exchange for capital freedom, while still maintaining the system's resistance to manipulation.
