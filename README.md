# Introduction

Derion is a permissionless AMM for compound-leverage perpetuals. Anyone can deploy a market for any index value — a Uniswap v3 pair by default, or any custom oracle feed — and trade leveraged Long/Short exposure on it with no liquidation price, no order book, and no operator.

### Decentralized Market

Anyone can create a market for any value feed and participate alongside or against everyone else, **with no privileged roles**.

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

A Derion market is split cleanly in two:

* **The pool is a pure trader engine.** Two sides, Long and Short, backed by a single reserve token. The pool holds the funds, enforces the pay-off curve, and verifies every state change. No liquidity provider lives inside it.
* **Liquidity is an external Vault.** An ownerless contract provides market depth by opening real Long and Short positions through the same public path every trader uses, and collects the funding paid by traders in return.

A pool works with or without the Vault. Absent liquidity, it is a pure trader-vs-trader market: positions deleverage instead of failing, and nothing ever freezes.

### Compounding Leverage

Unlike conventional perpetual and futures markets, Derion positions carry compound leverage: a position's value follows a power curve of the index price rather than a straight line. This is what removes the liquidation price — the losing side asymptotically approaches zero but never crosses it, so there is no margin call and no keeper racing to close positions in time. Traders benefit more from favorable moves and are penalized less by unfavorable ones, and they pay a continuous funding fee to the liquidity side for that exposure.

<figure><img src=".gitbook/assets/image (5).png" alt="" width="563"><figcaption></figcaption></figure>

The Vault and its depositors form the passive side of the market. They absorb the trader imbalance as counterparty and, in return, earn the funding paid by both Long and Short.

### Fully On-chain

Derion markets are fully automated smart contracts built on asymptotic power curves. There is no dependence on backend services for risk management, no liquidators, and no admin key over pool funds.

<figure><img src=".gitbook/assets/image (2) (1).png" alt="" width="563"><figcaption></figcaption></figure>

Derion pools are mathematically functional in any market condition, operating for unlimited periods of time with or without user interaction. No position is at risk of liquidation, nor can the market itself be bankrupted.
