---
description: Interest and premium, paid by traders to the LP class
---

# Funding

Funding on an exchange is a payment between longs and shorts every few hours, set by how far the perpetual trades from its index. Funding in a Derion pool is paid by traders to the pool's [LP class](../liquidity/lp-class.md), continuously, in two parts: **interest**, which every position pays on the reserve it holds, and **premium**, which the crowded side pays on its excess over the other side. Both are set per pool as half-lives. Both accrue to the LP class as growth of $$r_C$$ in place, with nothing to claim and no epoch to wait for.

| | Exchange funding | Derion funding |
| --- | --- | --- |
| Who pays whom | longs pay shorts, or the reverse | both sides pay the LP class |
| Rate | a premium index, reset every interval | two half-lives fixed in the pool config |
| Cadence | every 1 to 8 hours | continuous, accrued on every pool touch from the time elapsed |
| Quoted on | notional | position value; divide by $$k$$ to compare |
| Bookkeeping | debited and credited per account | the side reserves decay into the LP residual; no per-position ledger |

It is autonomous: computed from the pool state alone, with no keeper and no per-position bookkeeping. It runs entirely in reserve space and, apart from the protocol's cut, it never moves a token.

## Interest

Interest decays both side reserves proportionally:

$$
r_X \leftarrow r_X \cdot 2^{-\,\text{elapsed}\,/\,\text{INTEREST\_HL}} \qquad X \in \{A, B\}
$$

$$R$$ stays where it is, so the residual $$r_C = R - r_A - r_B$$ grows by exactly what the sides released. This is rent on reserve occupancy: every side pays the same rate on the reserve it holds, a matched Long and Short pays in full, and the LP class pays nothing because it is the recipient. Rounding is down, so traders pay at least the rate, and a non-empty side never decays to zero. `INTEREST_HL = 0` disables interest.

What a half-life means to a holder, as the share of position value gone per day at an unchanged price:

| `INTEREST_HL` | Daily bleed |
| --- | --- |
| 30 days | 2.3% |
| 90 days | 0.77% |
| 180 days | 0.38% |
| 365 days | 0.19% |

A position of value $$V$$ at power $$k$$ carries $$kV$$ of notional, so divide by $$k$$ to set it against exchange funding: a ×4 position on a 365-day half-life pays about 0.05% of notional a day. It is priced above exchange funding in a calm market because it is paying for something an exchange position does not have, convex P&L with no liquidation. [LP Economics](../liquidity/lp-economics.md) has the full table and the break-even the half-life is chosen against.

## Premium

Premium charges the crowded side. The gap between the two side reserves decays toward zero:

$$
\text{gap} = |r_A - r_B|,\qquad \text{paid} = \text{gap}\cdot\left(1 - 2^{-\,\text{elapsed}\,/\,\text{PREMIUM\_HL}}\right)
$$

The dominant side shrinks by that amount, into $$r_C$$ like interest; the smaller side is untouched. This is imbalance funding paid to the pool's counterparty, the model of pool-based perpetual exchanges rather than the peer model where longs pay shorts. Since the LP class holds no side, every side token belongs to a trader and the raw gap is the net trader imbalance; there is nothing to exclude. `PREMIUM_HL = 0` disables the premium.

For a trader this is the rate the interface shows as the premium rate: zero while your side is the smaller one, and rising with how far your side exceeds the other. For a provider it is the payment for carrying a one-sided book, which is where the LP class's exposure is largest.

{% hint style="info" %}
Both charges use exponential decay because it is the one live rate that composes exactly across touches: a pool poked twice reaches the same state as one poked once for the combined duration, at the same price. A linear rate would drift with poke frequency. Interest and premium each compose with themselves but not with each other, so poke cadence shifts a little of the incidence between the two charges. Both flows stay between the same three classes, so this moves who pays what, never whether value leaves the engine.
{% endhint %}

## The protocol cut

The only value that ever leaves the engine outside a settlement is the protocol's share of the funding release:

$$
\text{cut} = \frac{\text{release}}{\text{FEE\_RATE}},\qquad R \leftarrow R - \text{cut}
$$

It is carved out of $$R$$ at accrual with no transfer, so the trading path stays free of extra token movements, and sits in the balance as the outbox ($$\text{balance} - R$$) until a permissionless poke. `sync()` accrues funding at the TWAP, persists the state, and sends the outbox to `FEE_TO`. Donations to a pool land in the outbox and flush with it.

`FEE_TO` and `FEE_RATE` are immutables on the shared pool logic, set once at deployment, currently 1/5 of the funding release. `FEE_RATE = 0` means no cut; a poke then flushes only whatever was donated.

Because the LP yield accrues in place, poking is not a standing obligation for anyone. Nothing of the LP's is ever stranded outside the engine, and a pool left alone is worth exactly what a poked one is. The poke is a state refresh plus the protocol's own collection.

## Coefficient recovery

Funding runs on reserves, so the stored coefficients go stale on every accrual. A poke recovers each moved side once by inverting the curve and persists the result. A trade needs no persisted recovery because the Helper proposes fresh coefficients; when the oracle has diverged, the pool performs the same recovery transiently to price the spot-basis gates. Two corners exist at an extreme price relative to the mark. The inversion can be unrepresentable for a side; the pool then keeps that side's stored coefficient and the charge falls on the LP class instead, a dust-level concession. And a side's coefficient quantum can exceed its funded target, so the recovered side re-evaluates above where funding put it; if that leaves the two sides summing above the cut-reduced $$R$$, the pool reclaims the un-backable part of the cut so the stored state is always solvent.

The off-chain `View` contract mirrors both charges exactly, evaluated at the spot rather than the TWAP, so its quotes match what the next poke commits whenever the two bases agree.
