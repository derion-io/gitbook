# Price Oracle

### Uniswap v3

Uniswap v3 Oracle is supported for its availability and composability. Any token pair on the network is ready for its Asymptotic Power Perpetual pools.

<figure><img src="../.gitbook/assets/image (45).png" alt=""><figcaption><p>Uniswap V3 TWAP Oracle</p></figcaption></figure>

Uniswap Oracle provides both TWAP and SPOT prices. Derivable's double price system is another novel mechanism for derivative pricing as the price prediction is crucial and any market latency can be exploited.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Double Price System</p></figcaption></figure>

The double price system enforces the conversion rate to use the less beneficial result of the 2 prices. This system serves 2 purposes: (A) preventing price manipulation by arbitrageurs and (B) taking volatility fees for the Liquidity Provider in their unfavorable market.

The Liquidity Provider is always prioritized, ensuring the price is selected in their favor.

### Chainlink Feed

Derion leverages Chainlink oracle feeds for their **decentralized, reliable, and tamper-proof price data**. Chainlink's robust network of independent nodes aggregates data from numerous sources, eliminating single points of failure and ensuring high data integrity. This makes Chainlink the ideal choice for Derion's decentralized speculation markets, as it provides the **secure and accurate real-time price feeds** essential for fair and transparent market operations, aligning with Derion's core principle of eliminating privileged roles and backend dependencies.

With Chainlink Functions, Derion can create speculation markets for any real-world value or event, whether it's an election outcome or sports results.
