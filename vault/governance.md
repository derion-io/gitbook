# Strategy Governance

Which pools the Vault may fund — and each pool's ramp doubling time — is decided by the active **Strategy**: an immutable data blob whose on-chain pointer address is the strategy id. Governance is pure escrow accounting; no governance token is minted, and votes carry no claim on Vault capital.

### Strategies

`createStrategy(pools[], doublingTimes[])` permissionlessly writes a blob of 32-byte records, one per pool:

```
[ pool address : 20 bytes | doubling time : 12 bytes ]   — sorted ascending, unique
```

Lookups are O(1): callers supply a pool's index and the Vault verifies the record in place, so there is no on-chain search. A strategy is immutable by construction — changing policy means writing a new blob and moving the votes behind it.

### Voting

The vote weight is a locked LP position itself:

* `lock(strategy, pool, amount)` — escrow a served pool's LP token (`SIDE_C`) behind a candidate strategy; the locked amount *is* the vote. The pool must pin this Vault as its `PROVIDER`. Locking is pure escrow: the token keeps its full claim, and nothing about the pool's economics changes.
* `unlock(strategy, pool, amount)` — withdraw the vote and take the escrowed tokens back 1:1. Only the account that locked them can unlock them.
* `revote(from, to, pool, amount)` — re-point a vote to another candidate without un-escrowing.
* `changeStrategy(strategy)` — promote any blob holding a strict majority of all locked weight (`staked × 2 > totalLocked`) to active. The majority is the authorization; there is at most one active strategy at a time.

### Scope of the power

The strategy gates **funding only**. [Defunding is always open](depth-provision.md) — any pool dropped from the active strategy can be wound down by anyone — and no strategy can reach into a pool, touch trader collateral, or bypass a single pool gate. The worst a bad strategy can do is deploy depositors' capital badly, and that risk is priced at the [share boundary](deposit-withdraw.md).
