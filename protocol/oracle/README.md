---
description: Two prices, and why a trade never has to pick one
---

# Price Oracle

An exchange fills you from its order book and marks you at an index it computes. A Derion pool has no book, so the oracle is the whole price. Every pool names one price source in its immutable config: the [built-in fetcher](builtin.md) (a Uniswap v3 pair or a Chainlink feed), the [stock-token fetcher](stock-tokens.md), or any [custom fetcher](custom.md) contract. Whatever the source, it answers one question in one shape: two prices, a **TWAP** and a **spot**.

The TWAP is the mark. Funding accrues at it and positions are valued at it. The spot is the second price every trade must also clear. When the two agree, the usual state, you trade at that one price. When they disagree, your trade is priced at whichever of the two is worse for it, and the difference stays in the pool as income to the LP class.

### Two bases, no selection

Derivative pricing is prediction, and any gap between two live prices is exploitable. Earlier versions of the protocol resolved this by picking, per trade, the bound less favorable to the transactor. That required inferring the trade's direction before it was solved, and the declared slippage floors doubled as a direction declaration.

The current engine picks nothing. Every value gate of a [transition](../state-transition.md) runs twice, once at the TWAP basis and once at the spot basis, and the trade must pass both. Whichever bound is adverse to the trade binds automatically:

* the transactor's charge is required at both bases, and the dear one is the strict one;
* every class's per-share floor is required at both bases, so no incumbent loses at either.

Nothing has to be declared, a mis-declared direction buys nothing, and a diverged-oracle trade executes without any hint from the caller. When TWAP and spot agree, the second pass is skipped.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Double Price System</p></figcaption></figure>

### Which basis does what

The **TWAP** is the funding and marking basis. [Funding](../funding-rate.md) accrues at it (a poke is permissionless, so it must not let a manipulated spot set the funding price), positions are marked at it in events, and it anchors the class boundaries against single-block manipulation. The **spot** is the second gate basis only.

The gap between the two bases on a diverged trade is paid by the transactor, whose charge is at the adverse bound, and stays in the engine, where it accrues to the [LP class](../../liquidity/lp-class.md) at the non-binding basis. Incumbents on the sides are flat at their own binding bound and cannot collect it.

Concretely, while the two prices disagree a Long opens at the higher of them and closes at the lower, a Short opens at the lower and closes at the higher, and an LP deposit is priced at the higher of the class's two residuals and a withdrawal at the lower. That spread is the cost of trading while the sources disagree, and the pool's `getAmountOut` quote already includes it. A longer TWAP window makes the spread wider in fast markets and manipulation harder; the pool's creator picks that trade-off once, in the config.

### Caller-carried oracle data

A transition, a poke, and a quote each accept an optional `oracleData` blob that the pool forwards verbatim to the fetcher before reading the price. Read-only sources ignore it. A pull oracle uses it: the transactor brings a freshly signed price in their own transaction, the fetcher verifies the signatures and prices the trade off it. The fetcher decides what to accept, and signature verification there means a caller can make the price fresher, never wrong. A pool forwards native value to the fetcher to pay for a paid update; the fetcher spends at most what the update costs and refunds the rest. See [Custom Fetchers](custom.md).

### Hostile fetchers are contained

A pool's config is hashed into its address, so a manipulable or malicious fetcher can only ever harm the pool that chose it. Its risk never leaks into other pools or into the shared token. This is what lets liquidity be per market: a provider in one pool is never exposed to another pool's oracle, and nobody has to vet a listing on anyone else's behalf.
