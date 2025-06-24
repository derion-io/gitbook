# Introduction

Derion is a novel Decentralized Speculation Market that enables speculation on any value available via smart contract, including on-chain DEX prices or off-chain Oracle feeds.

### Decentralized Market

Anyone can initialize a speculation market for any value feed, allowing others to participate alongside or as counterparties, **with no privileged roles.**

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

The market features three distinct participant roles:

* **Long or Short:** Participants with positive gamma exposure, who pay an interest rate.
* **Liquidity:** Providers with negative gamma exposure, who earn the funding rate.

The market is designed to function effectively even with minimal or no liquidity. In the absence of counter-party liquidity, your position will incur no delta exposure and pay no funding fee. When a counter-party is present but significantly smaller than your market side, a corresponding fee rate will apply to a **partial** delta exposure.

### Compounding Leverage

Unlike Perpetual and Futures markets, Derion's Long/Short speculation positions utilize compound leverage, which generates the positive gamma exposure characteristic of its power curve, ensuring that **positions cannot be liquidated**. This is a desirable trait for traders, as it means they benefit more from favorable volatility and are less penalized by unfavorable movements; consequently, they pay a funding fee for the delta and gamma exposure to the Liquidity side.

<figure><img src=".gitbook/assets/image (5).png" alt="" width="563"><figcaption></figcaption></figure>

Liquidity Providers (LPs) represent the passive side of the market. They assume the counter-party risks associated with value movements and, in return, earn the funding fee from both Long and Short.

### Full On-chain

Derion markets are fully automated on a smart contract, utilizing dual asymptotic power curves. This design eliminates any dependence on backend services for risk management or position liquidation. The entire speculation market can be deployed on any smart-contract platform, and works completely decentralized without any privileged roles or backend services.

<figure><img src=".gitbook/assets/image (2).png" alt="" width="563"><figcaption></figcaption></figure>

Derion speculation pools are mathematically functional in any market condition, operating for unlimited periods of time with or without user interaction. No position is at risk of liquidation, nor can the market itself be bankrupted.
