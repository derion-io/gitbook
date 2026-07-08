---
description: The two-sided pool
---

# Trader Engine

A Derion pool is a pure trader engine. It holds a single ERC-20 reserve token, and its working balance — the **engine reserve** $$R$$ — is fully partitioned between the two sides by a single coefficient $$\alpha$$:

$$
q_A = \alpha x^k \qquad\quad r_A = \rho(q_A,\,R) \qquad\quad r_B = R - r_A
$$

There is no third side. No LP token, no idle tranche, no phantom counterparty, and no privileged operation touches the pool: liquidity, when present, is just another trader (see [Liquidity Vault](../vault/README.md)).

### Reserve and outbox

The pool's token balance is always at least $$R$$. The excess

$$
\text{outbox} = \text{balance} - R
$$

is the **funding outbox**: reserve that [funding](funding-rate.md) has already released to the liquidity provider but that has not been flushed out yet. The outbox backs nothing — it is excluded from every curve evaluation and can never re-enter the engine, because trades move $$R$$ and the balance in lockstep. It physically leaves the pool only on a poke.

### The whole book is three numbers

The entire pool state is the triple

$$
\langle\, R,\ \alpha,\ t \,\rangle
$$

— the engine reserve, the single Long coefficient, and the last funding-accrual time. Everything else — $$r_A$$, $$r_B$$, effective leverage — is re-derived from the [curve](pricing.md) on every touch. Positions are shares of a side, not accounts: there is no entry price, no per-position margin, and no funding bookkeeping to settle per user.

### A matched pair is a pure interest position

Holding the same fractional share $$f$$ of both sides is price-neutral: $$f\,r_A + f\,r_B = f\,R$$ at every price. The only force that moves a matched pair is funding — the decay of $$R$$ — so a matched holder pays exactly the funding rate to the liquidity provider. (Exactly price-neutral for a share-matched pair; approximately so for a value-matched pair away from balance.)
