# Introduction

Derion is a permissionless AMM for compound-leverage perpetuals. Anyone can deploy a market for any index value (a Uniswap v3 pair, a Chainlink feed, a tokenized stock, or any custom oracle) and trade leveraged Long/Short exposure on it with no liquidation price, no order book, and no operator.

### Decentralized Market

Anyone can create a market for any value feed and participate alongside or against everyone else, **with no privileged roles**.

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

A Derion market is one pool with three share classes, all issued as fungible tokens:

* **Long** and **Short**, each a true power pay-off of the index price. Traders hold these.
* **The LP class**, the reserve the two pay-offs leave over. It is the counterparty to both sides, the depth that keeps them at full leverage, and the recipient of every fee and funding stream, less the protocol's cut of funding. Anyone can hold it, directly or through the shared [Liquidity Vault](liquidity/vault.md).

The pool holds the funds, enforces the pay-off curves, and verifies every state change against both of its oracle's prices. No address is special to it: providing liquidity is a trade through the same path as every other trade, and the pool works with an empty LP class just as it works with a deep one.

### Compounding Leverage

Unlike conventional perpetual and futures markets, Derion positions carry compound leverage: a position's value follows a power curve of the index price rather than a straight line. This is what removes the liquidation price. The losing side approaches zero asymptotically but never crosses it, so there is no margin call and no keeper racing to close positions in time. Traders gain more from favorable moves and lose less on unfavorable ones, and they pay a continuous funding fee to the LP class for that exposure.

<figure><img src=".gitbook/assets/image (5).png" alt="" width="563"><figcaption></figcaption></figure>

The LP class is the passive side of the market. It absorbs the trader imbalance as counterparty and, in return, collects the funding paid by both Long and Short (less the protocol's cut), the opening fees, and the spread between the oracle's two prices.

### Fully On-chain

Derion markets are fully automated smart contracts built on asymptotic power curves. There is no dependence on backend services for risk management, no liquidators, and no admin key over pool funds.

<figure><img src=".gitbook/assets/image (2) (1).png" alt="" width="563"><figcaption></figcaption></figure>

Derion pools are mathematically functional in any market condition, operating for unlimited periods of time with or without user interaction. No position is at risk of liquidation, nor can the market itself be bankrupted.
