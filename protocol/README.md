---
description: How a Derion pool works, for perpetual traders, perp-DEX liquidity providers, and AMM liquidity providers
---

# Protocol Design

Derion is a set of immutable smart contracts for permissionless leveraged-perpetual markets on any index value, with any leverage, reserved in any ERC-20 token. A market is one pool. The pool holds one reserve token and issues three fungible share classes against it: a **Long** and a **Short**, each a true power pay-off of the index price, and an **LP class** that owns whatever reserve the two pay-offs leave. Every open, close, deposit, and withdrawal goes through the same public call, is checked at both of the oracle's prices, and settles against the pool's own reserve. Nothing else is involved: no order book, no keeper, no operator.

Most readers arrive knowing one of three systems well. This page maps each onto a Derion pool. The pages after it work through the mechanics.

### If you trade perpetuals

The vocabulary carries over. You go Long or Short at a leverage, you pay funding while you hold, you are marked at an oracle price, and you close whenever you like. What changes is what the position is and how it pays.

| | Exchange perpetual | Derion pool |
| --- | --- | --- |
| Position | an account with an entry price and a margin balance | a token: shares of one common pay-off, with no entry price |
| Leverage | fixed notional, linear P&L | compound: a Long ×k is worth $$p^k$$, a Short ×k is worth $$p^{-k}$$ |
| Liquidation | at a price, by keepers, backed by an insurance fund and auto-deleveraging | none; a side that outgrows the pool delevers smoothly and never crosses zero |
| Funding | longs pay shorts or the reverse, every few hours | both sides pay the pool continuously: interest on the reserve held, plus a premium on the crowded side |
| Price | order book, marked at an index | the oracle's TWAP and spot; whichever is worse for your trade binds |
| Counterparty | other traders and the exchange's insurance fund | the pool's LP class |
| Fees | taker fee on open and close | an optional opening fee, set per pool; closing is free |

Three things are worth knowing before the first trade. Compounding cuts your way in both directions: at ×4 a 10% move up pays 46%, a 10% move down costs 34%, and a 25% move against you leaves about a third of the position rather than liquidating it ([Pay-off Curve](pricing.md)). Funding is priced to pay the LP for that convexity, so it runs above exchange funding in a calm market; [Funding](funding-rate.md) shows how to compare the two on a notional basis. And a side that grows past half the pool's reserve starts to delever: its gains flatten until more liquidity arrives, which the interface shows as deleverage risk and [Three Share Classes](engine.md) explains.

### If you provide liquidity on a perpetual DEX

The LP class is the counterparty pool, the role a GLP- or HLP-style vault plays, with one difference in kind: it is a share class of a single market, not a shared basket. One pool, one reserve token, one oracle. A bad oracle or a toxic market can only hurt the LPs of the pool that chose it.

What you earn is what traders pay: the interest both sides pay on their reserve, the premium the crowded side pays, the opening fee if the pool charges one, and the gap between the oracle's two prices on every trade executed while they disagree, less the protocol's cut of funding (one fifth at the deployed rate). None of it is streamed or claimed. It accrues as growth in the reserve behind each LP share ([Funding](funding-rate.md), [Opening Fee](opening-fee.md)).

What you carry is trader P&L, with a shape you can price. The LP class owns the reserve the two power curves leave, which is largest at the price center and shrinks as the price moves either way: a short straddle on $$p^k$$, whose cost per unit of trader reserve grows with $$k^2\sigma^2$$. It recovers fully on a round trip and loses on a trend in either direction. [LP Economics](../liquidity/lp-economics.md) gives the break-even and [Pool Parameters](../guide/pool-parameters.md) turns it into config values.

What you do not carry is the tail risk of a linear book. A winner's claim approaches the pool's reserve and never reaches it, so there is no bad debt, no insurance fund to backstop, no auto-deleveraging that closes winning positions, and no open-interest cap to administer. Solvency is one inequality the pool checks on every state change ([Three Share Classes](engine.md)), and no other account's trade can lower your value per share at either oracle price ([Value Gates](value-invariant.md)). There is no privileged role either: you hold the LP class directly, or through the shared [Liquidity Vault](../liquidity/vault.md), which is one holder among any others and rebalances through the same public path as every trade.

### If you provide liquidity on an AMM

The LP class behaves like a pool token. It is an ERC-1155 id of the pool, worth a pro-rata share of the reserve the two curves leave. Income accrues to the pool and shows up as reserve per share, the way fees do in a Uniswap v2 pair; nothing is collected separately. You deposit and withdraw through the pool itself, transfer the position, or hold it in a contract. Pools are created permissionlessly, and a configuration maps to exactly one pool address, so a pool is found from its parameters the way a Uniswap v3 pool is found from its tokens and fee tier.

The exposure differs from swap liquidity in where the counterparty sits. There are no swaps to be the other side of, so there is no impermanent loss to arbitrage. In its place is the trader book: the LP class collects funding and loses when the price moves away from the center, in either direction. It is the same short-convexity profile as impermanent loss, with funding standing where swap fees do ([Three Share Classes](engine.md), [LP Economics](../liquidity/lp-economics.md)).

Entry and exit carry a protective spread rather than a fee. A deposit is priced at the higher of the LP class's two oracle bounds and a withdrawal at the lower, so a round trip through the class pays a spread rather than earning one, and when the two prices agree there is no spread at all. Funding accrues before any deposit is priced, so it cannot be back-run ([The LP Class](../liquidity/lp-class.md)).

A Derion pool is also a hedge for the swap liquidity you already hold. Equal reserve in a pool's Long and Short is a long straddle on $$p^k$$: it gains on any move and pays funding for it, which is the mirror of impermanent loss ([AMM-LP IL Hedge](../apps/amm-lp-il-hedge.md)).

### The pages

* [Pay-off Curve](pricing.md): the one curve that prices everything, why it compounds, and why it removes the liquidation price.
* [Three Share Classes](engine.md): what a Long, a Short, and an LP share are, the four numbers that make up a pool's whole book, and the one inequality that is solvency.
* [Price Oracle](oracle/README.md): the two prices every pool trades at and why nothing has to be declared. Sources: [Uniswap v3 and Chainlink](oracle/builtin.md), [tokenized stocks](oracle/stock-tokens.md), [custom fetchers](oracle/custom.md).
* [State Transition](state-transition.md): what happens when you open, close, flip, deposit, or withdraw, and how the pool checks a trade it did not compute.
* [Value Gates](value-invariant.md): the rules that keep every class's value per share from falling on anyone else's trade.
* [Funding](funding-rate.md): interest and premium, who pays, who receives, and how to compare with exchange funding.
* [Opening Fee](opening-fee.md): the one optional fee, where it lands, and when a pool should charge it.
