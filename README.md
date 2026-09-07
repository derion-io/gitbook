---
description: Permissionless perpetual AMM with compound leverage and no position liquidation
---

# Introduction

Derion is a permissionless perpetual AMM with compound leverage and no position liquidation. Anyone can deploy a market on any index value with an oracle feed, trade leveraged Long or Short exposure on it, or provide the liquidity behind it. Positions are fungible tokens whose value follows a power of the index price rather than a straight line. That curve is what removes the liquidation price, and the pool behind it runs with no order book, no operator, and no key over its funds.

### Permissionless

A Derion market is one contract, deployed by whoever wants it. The deployer picks a reserve token, a price source, a leverage, and the funding and fee rates; the configuration is hashed into the pool's address, so the same market cannot be deployed twice and anyone can derive its address from its parameters. The price source can be a Uniswap v3 pair, a Chainlink feed, a tokenized stock, or any custom fetcher contract. Creating a pool earns nothing and grants no privilege over it.

After that there is no operator. Nobody can list or delist the market, change its parameters, or decide who trades. Providing liquidity goes through the same public path as every trade, so there is no provider role to be granted either. No admin, pause, or upgrade key touches pool funds anywhere in the system.

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

### A perpetual AMM

There is no order book, no matching, and no counterparty to find. The pool takes the other side of every trade by formula. It holds a single ERC-20 reserve and issues three fungible share classes against it: **Long**, **Short**, and the **LP class**. Opening a position mints shares of a side out of the reserve; closing burns them back into it. The index price comes from the pool's oracle, and two pay-off curves decide how much of the reserve each class is worth at that price. Every trade is verified at both of the oracle's prices, the time-weighted average and the spot, so whichever is adverse to the trade binds.

Positions are shares of a class, not accounts. They have no entry price, no expiry, and no per-account margin. Two holders of the same side own the same pay-off, and a position can be transferred, held by a contract, or composed like any other token.

The LP class is the counterparty. It holds whatever the two curves leave of the reserve, which makes it both the depth that keeps the sides at full leverage and the balance sheet that absorbs their imbalance. For that it collects the continuous funding both Long and Short pay, less the protocol's cut, along with the opening fee and the spread between the oracle's two prices. Anyone can hold it, directly or through the shared [Liquidity Vault](liquidity/vault.md). A pool with an empty LP class still works: it deleverages sooner, and it never fails.

### Compound leverage

A Derion position does not track the index price along a straight line. A Long ×3 is worth the cube of the index price and a Short ×3 its inverse cube, so the leverage a trader experiences, the percentage change in value per percentage change in price, stays at 3 as the price moves, until the side grows toward the pool's reserve and begins to delever. A linear pay-off has its stated leverage only at the entry price: it fades as the position wins and grows without bound as it loses.

Compounding works in the trader's favor in both directions. A Long ×4 opened with 1.0 of reserve is worth about 1.46 after the index rises 10% and about 0.66 after it falls 10%; a linear ×4 would pay 1.40 and 0.60. Gains accelerate on the right side and losses slow on the wrong side, and what a trader pays for that shape is a continuous funding fee to the LP class.

<figure><img src=".gitbook/assets/image (5).png" alt="" width="563"><figcaption></figcaption></figure>

### No position liquidation

A linear pay-off crosses zero at a finite price, and that crossing is a liquidation price. Every market built on one depends on keepers closing positions before it is reached, and on an insurance fund, auto-deleveraging, or socialized losses for when they are late.

A power curve never crosses zero. On the losing side, value approaches zero and never reaches it, so there is nothing to liquidate and no margin call to answer. On the winning side, the claim bends onto an asymptote as it grows toward the pool's reserve and approaches it without ever taking all of it; effective leverage compresses smoothly toward zero instead. Deleveraging replaces liquidation, so there are no keepers, no bad debt, and no one racing an oracle update.

<figure><img src=".gitbook/assets/image (2) (1).png" alt="" width="563"><figcaption></figcaption></figure>

Solvency is a property of the arithmetic, not an operational process. It reduces to one inequality on the pool's state, checked at every state change and holding at every price in between, in a pool untouched for a year as much as in one traded every block. No position can be liquidated, and the market itself cannot go bankrupt.

[Protocol Design](protocol/README.md) works through the curve, the three share classes, the value gates, and funding. [Liquidity](liquidity/README.md) covers the LP class and the Vault, [Guide](guide/README.md) covers trading and pool creation, and [Contracts](contracts/README.md) has the addresses and APIs.
