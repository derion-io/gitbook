# Price Oracle

Uniswap v3 Oracle is chosen for its availability and composability. Any token pair on the network is ready for its Asymptotic Power Perpetual pools.

<figure><img src="../.gitbook/assets/image (45).png" alt=""><figcaption><p>Uniswap V3 TWAP Oracle</p></figcaption></figure>

Uniswap Oracle provides both TWAP and SPOT prices. Derivable's double price system is another novel mechanism for derivative pricing as the price prediction is crucial and any market latency can be exploited.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Double Price System</p></figcaption></figure>

The double price system enforces the conversion rate to use the less beneficial result of the 2 prices. This system serves 2 purposes: (A) preventing price manipulation by arbitrageurs and (B) taking volatility fees for the Liquidity Provider in their unfavorable market.

The Liquidity Provider is always prioritized, ensuring the price is selected in their favor.
