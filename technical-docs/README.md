# Technical Docs

{% embed url="https://docs.google.com/presentation/d/1mNTU396m3-69i-4chvVlmBgbxOZi8rL3ObF8KVfF6QE/" %}
Presentation Slide
{% endembed %}

## Basic Concepts

CEX: Centralized Exchange. E.g. Binance, Coinbase, FTX, Mt. Gox, etc.

DEX: Decentralized Exchange. E.g. Uniswap, PancakeSwap, KyberSwap, etc.

Token Price (P) is the market price of one token (base) to another (quote). P is fetched either from off-chain CEXes, or on-chain DEXes. ETH/USD price is the conversion rate of ETH to US Dollars, and it must be fetched from an off-chain CEX. ETH/USDC is the conversion rate of ETH to USDC token, and it can be fetched from an on-chain DEX or off-chain CEX. ETH/USDC and ETH/USD are not the same.

Power Perpetual (PP): Derivatives that track an asset price in the form of $${P\over M}^x$$where $$P$$ is a Token Price and M is a Mark Price.

Mark Price (M) in Derivable is initialized with the first value of P being fetched, and will not follow the market price after that. Mark Price will be changed by internal mechanisms of the Power Perpetual protocol in a deterministic way.

E.g. A Power Perpetual token $${ETH\over USDC}^4$$ has its M initialized with 1500, when the market price of ETH/USDC changes to 1600, the price of the token will be $${1600\over 1500}^4 = 1.294538272$$.

Exposure: how many times the derivative token value will be changed if the underlining price changes a fixed reference rate of price (+1%). In the example above, the exposure of the derivative token is $$e = ({1616\over 1500}^4-{1600\over 1500}^4)  ≈ 5.26\%$$.

Liquidity Provider Token (LP): the token received when the liquidity is added to an AMM pool.

Collateral Provider Token (CP): the token received when the collateral is added to a Derivable DDL pool.

Collateral Ratio (CR): the ratio of the total token reserve value over the value of the total derivative. Unlike other stablecoins/synthetic protocols, Derivable does not assume or require the CR > 1 property (over-collateralization).

Funding Rate (Exposure Fee): the value the derivative token holder must pay while holding the derivative tokens. The Funding Rate is charged constantly over time. The Funding Rate is different between the Long and Short derivatives and depends on how balanced the two sides are. The less balanced the two sides, the higher the funding rate. The lower the CR value is, the higher the funding rate is.

Deleverage: the action of re-balance the leverage of the DDL pool, to reuse an under-collateralized pool. This action can be done by anyone when the pool is under collateralization.
