# Token Conversion

Conversion rates between each token are calculated based on the input price. $$E^x_{S.T}$$ = f(S, T, x) where S is the source token, T is the target token, and x is the input target price. x can be omitted when the current market price is used.

Uniswap Oracle provides both spot price and TWAP. For most parts of the protocol, the TWAP price is used for its manipulation resistance. Only for token conversion, the double price system is utilized since the price prediction is crucial so any market latency can be manipulated. Unlike in an AMM, arbitrageurs are not necessary nor desired in Power Perpetual.

<figure><img src="https://lh3.googleusercontent.com/FbZZKg0A9DrkP2o5AL-K_8_bmHRzQm1BnR2tnir6iASEyvYT9dGf9y0l6PIzUoAi3Y7pi8BtYzn1-L2EhIbbUEVLSvWCHh4KdJQBzo7BSfJHSp3OwI69HFtAxDDW4IwILSb_C50WQhKdo-BXKreS0Z_yfCMRKZTieLU65WrVGN_FiLVNGb5q4b9b8rmUpw" alt=""><figcaption><p>Double Prices System</p></figcaption></figure>

The double price system enforces the conversion rate to use the less beneficial result of the 2 prices. This system serves 2 purposes: (A) preventing price manipulation by arbitrageurs, and (B) taking volatility fees for the Collateral Provider in their unfavorable market.

With:

* $$E^S_{DiC}$$ is the exchange rate from Di to C using the Spot Price
* $$E^T_{DiC}$$ is the exchange rate from Di to C using the TWAP Price

We have the conversion rate between C and D tokens as follows:

## Mint Derivative

Derivative token D can be minted by locking the same market value of C as Derivative Collateral.

* $$\Delta_{Di} = \Delta_C \div max(E^S_{DiC}, E^T_{DiC})$$

## Burn Derivative

Collateral can be unlocked from Derivative Collateral by burning the same market value of D.

* $$\Delta_C = \Delta_D \times min(E^S_{DiC}, E^T_{DiC})$$
