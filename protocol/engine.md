---
description: Long, Short, and the LP class
---

# Three Share Classes

A Derion pool holds a single ERC-20 reserve token and issues three tokens against it. To a trader, the **Long** and the **Short** are the positions. To a liquidity provider, the **LP class** is the deposit. All three are shares of the same reserve. The pool's working balance, the **engine reserve** $$R$$, is split between them at every touch by the two [curves](pricing.md):

$$
r_A=\rho(\alpha x^{k},R) \qquad\quad r_B=\rho(\beta x^{-k},R) \qquad\quad r_C=R-r_A-r_B
$$

Each class is an ERC-1155 token of the pool, its id the side code (Long `0x10`, Short `0x20`, LP `0x30`) packed above the pool address, minted and burned through the same `transition()` path, and each is worth its reserve pro-rata: a holder of $$n$$ out of $$s_X$$ shares owns $$n\,r_X/s_X$$.

<figure><img src="../.gitbook/assets/image (2) (1).png" alt="" width="563"><figcaption></figcaption></figure>

### What a trader holds

Positions are shares of a class, not accounts. A Long is the same token for everyone who holds it, whenever they bought, so there is no entry price, no per-position margin, and no per-user funding ledger. Your P&L is the change in the class's reserve per share since you bought, and over the same period it is the same percentage for every holder of that class. Adding to a position mints more of the same token. Closing burns some or all of it, or is done by transferring the token into the pool. The position can sit in a wallet, in a contract, or on a secondary market, and it keeps working while it does.

### The whole book is four numbers

$$
\langle\, R,\ \alpha,\ \beta,\ t \,\rangle
$$

The engine reserve, the two coefficients, and the last funding-accrual time. Everything else, the three reserves and every effective leverage, is re-derived from the curves on every touch at the oracle price.

### What a liquidity provider holds

$$r_C$$ has no coefficient. Its reserve is whatever the two power curves leave, which gives it a definite pay-off of its own: peaked at the price center and vanishing toward both extremes. To a perp-DEX depositor it is the counterparty pool; to an AMM provider it is the pool token. It is a short-volatility tranche, and it plays four roles at once.

**It is the counterparty.** As price moves, the winning side's reserve grows faster than the losing side's shrinks, and the difference breathes out of $$r_C$$. Given a state $$(R, \alpha, \beta)$$, $$r_C$$ is a pure function of price: a price round trip restores it exactly, with no path dependence.

**It is depth, in both directions.** The inflection of $$\rho$$ sits at $$R/2$$ and $$R$$ carries the residual, so a pool with a funded LP class deleverages later on both sides at once. There is no minor side to pick and nothing to rotate.

**It collects every revenue stream, in-engine.** [Funding](funding-rate.md) decays the sides into it; the [opening fee](opening-fee.md) is forced onto it by its fee-raised floor; the gap between the two oracle bounds on a diverged trade lands on it too. Nothing is routed, streamed, or flushed to a provider address. Value arrives as growth of $$r_C / s_C$$, the way swap fees grow the reserves behind a constant-product LP token.

**It is open.** Anyone deposits reserve for LP shares and withdraws through `transition()`, priced at the class's adverse bound in each direction exactly like the sides. Ownership is the token. The pool knows no liquidity provider by name; the [Vault](../liquidity/vault.md) is one holder among any others. See [The LP Class](../liquidity/lp-class.md).

### Solvency is one inequality

The first question a provider asks of a counterparty pool is whether traders can win more than it holds. Here the answer is arithmetic. Because $$(\alpha x^k)(\beta x^{-k}) = \alpha\beta$$ does not depend on price, three statements are equivalent:

$$
\alpha\beta \le \left(\tfrac{R}{2}\right)^2
\quad\Longleftrightarrow\quad
r_A + r_B \le R \ \text{ at every price}
\quad\Longleftrightarrow\quad
r_C \ge 0 \ \text{ at any one price}
$$

The right-hand equivalence is the load-bearing one: checking the residual at the current price is a global solvency proof. The pool therefore needs no separate product check, no open-interest cap, and no insurance fund. Initialization rejects a seed with a negative residual, and the per-share floor on the LP class ([Value Gates](value-invariant.md)) keeps every later state solvent. That floor is stronger than $$r_C \ge 0$$: it is per-share non-decreasing, and the dead shares minted at initialization make the supply permanent, so a solvent pool cannot be ground insolvent.

### Initialization

A pool is seeded with $$\langle R, \alpha, \beta\rangle$$ and the reserve $$R$$ paid in. All three classes must clear a minimum reserve, which also rejects an insolvent seed outright, and all three seeds are minted to an address nobody controls. They protect against share-inflation attacks, and they are the permanent non-zero supplies the per-share gates divide by. The seeding price is the oracle's spot alone: only the creator's own dead shares are being split, so a mispriced spot mis-splits the creator's deposit and harms nobody else.

### Reserve and outbox

The pool's token balance is always at least $$R$$. The excess

$$
\text{outbox} = \text{balance} - R
$$

holds only the protocol's cut of [funding](funding-rate.md), plus any donations, waiting for a poke to flush it. It backs nothing, is excluded from every curve evaluation, and is invariant across trades: a transition moves $$R$$ and the balance in lockstep, so nothing a trader does can feed or drain it.

### A matched pair is a straddle

Holding the same fractional share of Long and Short is not price-neutral in a two-curve engine. Below the inflection, $$r_A + r_B = \alpha x^k + \beta x^{-k}$$ is smallest at the price center and grows in either direction, so a matched pair is a long straddle on $$p^k$$: it gains on any move and pays [funding](funding-rate.md) for the privilege. The LP class is the other side of that trade, which is what [LP Economics](../liquidity/lp-economics.md) prices. For a provider of constant-product AMM liquidity, whose impermanent loss has the opposite shape, the pair is a hedge ([AMM-LP IL Hedge](../apps/amm-lp-il-hedge.md)).
