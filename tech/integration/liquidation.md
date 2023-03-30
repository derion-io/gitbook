# Price Oracle

Uniswap v3 Oracle is chosen for its availability and composability. Any token pair on the network is ready for its Asymptotic Power Perpetual pools.

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption><p>Uniswap V3 TWAP Oracle</p></figcaption></figure>

Uniswap Oracle provides both TWAP and SPOT prices. The double price system is utilized only for token conversion since the price prediction is crucial so any market latency can be manipulated. Unlike in an AMM, arbitrageurs are not necessary nor desired in Derivable.

<figure><img src="https://lh3.googleusercontent.com/FbZZKg0A9DrkP2o5AL-K_8_bmHRzQm1BnR2tnir6iASEyvYT9dGf9y0l6PIzUoAi3Y7pi8BtYzn1-L2EhIbbUEVLSvWCHh4KdJQBzo7BSfJHSp3OwI69HFtAxDDW4IwILSb_C50WQhKdo-BXKreS0Z_yfCMRKZTieLU65WrVGN_FiLVNGb5q4b9b8rmUpw" alt=""><figcaption><p>Double Prices System</p></figcaption></figure>

The double price system enforces the conversion rate to use the less beneficial result of the 2 prices. This system serves 2 purposes: (A) preventing price manipulation by arbitrageurs, and (B) taking volatility fees for the Liquidity Provider in their unfavorable market.
