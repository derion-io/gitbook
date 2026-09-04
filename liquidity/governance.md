# Strategy Governance

Which pools the Vault may fund, and each pool's ramp and headroom, is decided by the active **Strategy**: an immutable data blob whose on-chain pointer address is the strategy id. Governance is pure escrow accounting over the Vault's own shares. No separate token is minted.

### Strategies

`createStrategy(pools[], rampHLs[], headRooms[])` permissionlessly writes a blob of 32-byte records, one per pool, after a header holding the creation time:

```
[ pool : 20 bytes | ramp half-life : 8 bytes | headroom (Q16) : 4 bytes ]   sorted strictly ascending by pool
```

Pools must be sorted, unique, and non-zero, and each ramp half-life positive. A strategy funds at most 32 pools, because deposits and withdrawals walk the whole active set and the walk has to fit in a block. Lookups are O(1): callers supply a pool's index and the Vault verifies the record in place.

`extendStrategy(base, pool, rampHL, headRoom)` copies `base` and splices one pool into its sorted slot, producing a new blob. `Factory.deployWithStrategy` chains this with a pool deployment, so a pool can be deployed, seeded, and registered under an existing policy in one transaction. The new strategy still has to win the vote.

A strategy is immutable by construction. Changing policy means writing a new blob and moving the votes behind it. Only blobs created through the Vault are registered; a hand-deployed byte-compatible blob can be neither voted for nor promoted, so the Vault never reads a header it did not stamp.

### Voting

The vote is the Vault's own share, escrowed:

* `lock(strategy, amount)` escrows VaultLP behind a candidate. The shares move by internal transfer (no approval), keep their full NAV claim, return 1:1 on unlock, and cannot be withdrawn while locked.
* `unlock(strategy, amount)` withdraws the vote. Only the account that locked can unlock.
* `revote(from, to, amount)` re-points weight to another candidate without un-escrowing.
* `changeStrategy(strategy)` promotes a candidate holding a strict majority of the vote to active. The majority is the authorization; there is at most one active strategy.
* `lockAndVote(strategy, amount)` locks and requires the strategy to be active afterward, reverting otherwise. Use it to fail loudly rather than land a silent non-promoting vote.

**Weight is time at risk.** A lock's weight is its amount multiplied by the time since it was locked, so capital that has been committed longer counts for more. A fresh lock has zero weight in its own block. That kills flash and genesis capture: a strategy can only be promoted once its weight out-ages the rest, never by a same-block deposit-and-seize. Seniority travels with a revote, since it is a property of the committed capital and not of the candidate, so a revote of aged weight can cross the majority instantly; a plain lock cannot, and its crossing surfaces later through `changeStrategy` as the weight accrues. Unlocking removes the proportional share of a lot's seniority, so what remains keeps its weight.

### Defund first

A strategy cannot go active while the Vault still holds a position in a pool it drops. Promotion reverts until those pools are wound down. The pools an incoming strategy drops are still in the active set until the flip, so the active blob cannot prove them unfunded; the candidate itself, once it holds the majority, is the authority for `defund` on them ([Depth Provision](depth-provision.md)). For that purpose alone the majority is counted on raw locked amounts rather than time-weighted, so a fresh majority can start winding down before its weight has aged. The consequence is that the Vault's positions are always a subset of the active set, so the NAV walk over active pools covers the whole book and no dropped-but-held position can sit outside NAV.

On promotion the Vault records the time and announces every pool entering or leaving the active set (`VaultPool(pool, active)`, in ascending pool order), so indexers can maintain the fundable set from logs alone. The promotion time is also the ramp anchor for any pool that has not yet traded under the Vault.

### Scope of the power

The strategy gates funding only. Defunding is always open; no strategy can reach into a pool, touch trader collateral, or bypass a pool gate; and every Vault trade is floored at the pool's own adverse quote. The worst a bad strategy can do is deploy depositors' capital badly, into a pool that then loses, and only at the ramp's pace, which is bounded below by `MIN_RAMP_HL`. That risk is priced at the [share boundary](deposit-withdraw.md).

Vote capture costs real capital: a majority of the time-weighted VaultLP supply, held long enough to out-age the rest, all of it at NAV and at risk in the same book it steers. Standard token-governance levers (a timelock, a quorum, a per-holder cap) can be layered on if a deployment warrants.
