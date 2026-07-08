# Funding

Funding is what traders pay for their exposure, and what the liquidity provider earns for absorbing it. Unlike conventional perpetual exchanges with periodic funding epochs, Derion's funding is autonomous and continuous: it accrues on every pool touch from the time elapsed since the last one, computed purely from the pool state — no keeper, no schedule, no per-position bookkeeping.

There are two components, each configured as a half-life.

## Interest

Interest decays the engine reserve:

$$
R \leftarrow R \cdot 2^{-\text{elapsed}\,/\,\text{INTEREST\_HL}}
$$

The coefficient $$\alpha$$ — every position's price-tracking identity — is left untouched, and the side reserves are re-derived from the [curve](pricing.md). Decaying $$R$$ rather than the positions makes interest *rent on reserve occupancy*: a saturated (dominant) side, whose reserve scales with $$R$$, pays the bulk of it; a small side deep on its power branch pays almost nothing until the shrinking inflection point reaches it; a matched Long+Short pair pays in full. `INTEREST_HL = 0` disables interest.

## Premium

Premium charges the crowded side. The gap between the **trader-held** reserves of the two sides decays toward zero:

$$
\text{gap} = |t_A - t_B|,\qquad \text{paid} = \text{gap}\cdot\left(1 - 2^{-\text{elapsed}\,/\,\text{PREMIUM\_HL}}\right)
$$

The dominant trader side shrinks by that amount (and $$R$$ with it, keeping $$r_A + r_B = R$$ exact); the smaller side is untouched. This is imbalance funding accrued to the liquidity side — the model of pool-based perp DEXs — not the CEX model where longs pay shorts: the side creating the counterparty risk pays the party absorbing it.

"Trader-held" means the [Vault](../vault/README.md)'s own position is excluded from the gap. The premium must track the liquidity provider's actual net counterparty risk, not the depth the Vault itself provides — otherwise the Vault's own depth would damp the charge, or even misdirect it onto the provider's own side. `PREMIUM_HL = 0` disables the premium.

{% hint style="info" %}
Both charges use exponential decay because it is the unique live basis that composes exactly across touches: a pool poked twice reaches the same state as one poked once for the combined duration (at the same price). A linear rate would drift with poke frequency. Left completely untouched, the premium self-balances the pool toward $$t_A = t_B$$.
{% endhint %}

## The outbox and the flush

Funding releases reserve but transfers nothing on the trading path — the released reserve accumulates in the pool balance as the **outbox** (`balance − R`), keeping swap gas deterministic and free of extra token transfers. It physically leaves the pool only on a poke:

```
every touch:    R shrinks by (interest + premium); the outbox grows by the same amount
sync():         pending = balance − R
                fee     = pending / FEE_RATE     → FEE_TO      (protocol cut)
                payout  = pending − fee          → PROVIDER    (the LP yield)
```

`sync()` is permissionless: it accrues funding at the manipulation-resistant TWAP (no trade follows, so there is no adverse bound to pick), persists the state, and flushes the outbox. Anyone can poke; the [Vault](../vault/depth-provision.md) exposes batch pokes over every pool it serves.

* `FEE_TO` and `FEE_RATE` are immutables on the shared pool logic, set once at deployment — currently 1/5 of the flushed funding.
* `PROVIDER` is a per-pool config payee, never a permission. With `PROVIDER = 0`, the LP yield routes to `FEE_TO`.
* Donations to a pool land in the outbox and flush with it.

The off-chain `View` contract mirrors both charges exactly, so quotes track the pool byte-for-byte between pokes.
