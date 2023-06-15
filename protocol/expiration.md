# Maturity

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption><p>Position Maturity</p></figcaption></figure>

All Derivable positions have a maturity period during which the position's payoff is lower than its actual value. The maturity duration can be configured by the pool, which creates a soft-lock effect for newly opened positions and protects markets with low-cap, high-volatility indexes from price manipulation attacks.

Once the maturity time has elapsed, the position becomes fully matured and fully fungible. However, until that time, the position is only partly fungible and subject to the following rules:

* A maturing position can be divided into smaller positions with the same maturity time.
* A maturing position cannot be transferred or merged into a less matured position.
* Merging two positions will result in a position with a later maturity time.

Closing a maturing position will result in a payoff of zero or less than the position's value, which is calculated as follows:

$$
v\times(1-256^{({{T-t}\over M}-1)})
$$

Where:

* $$v$$ = the position's value when fully matured
* $$T$$ = the maturity time of the position
* $$M$$= the maturity duration configuration of the pool
* $$t$$ = the current `block.timestamp`
