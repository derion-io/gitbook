# Technical Design

Derion comprises a set of well-isolated, immutable smart contracts, designed for deployment on any EVM-compatible chain.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### One logic contract, one proxy per pool

A single **Pool Logic** contract carries all the code: the curves, the funding, the transition verification. Each pool is a separate **ERC-3448 MetaProxy** deployed with CREATE2, delegatecalling that shared logic. The pool's immutable `Config` is baked into the proxy bytecode itself and forwarded on every call, so reading it costs no storage access; only the mutable state (the reserve, the two coefficients, the funding clock) lives in storage.

**Config is the address.** The CREATE2 address hashes the init code, which embeds the encoded config, so a configuration maps deterministically to exactly one pool address. The same config cannot be deployed twice, and anyone can derive a pool's address from its config alone. Pool creation is permissionless and automatically de-duplicated. Changing the `Config` struct re-keys every pool, which is why successive protocol versions occupy disjoint address spaces.

### Why per-pool proxies, not a singleton

Every pool runs the same logic, so isolation is not about code bugs; it is about custody. Each proxy holds only its own reserve token, which bounds any exploit to a single pool's reserve and forces an attacker to re-run it pool by pool. Decisively, a **hostile configuration** (a malicious fetcher, a manipulable oracle, or an exotic fee-on-transfer reserve token) can harm only the pool that chose it. A singleton would commingle every pool's reserve in one pot.

### Trust tiers

| Tier | Contract | Can | Cannot |
| ---- | -------- | --- | ------ |
| law + custody | Pool Logic + proxy (per pool) | hold funds; enforce the curves and the value gates; mint/burn its three classes | none |
| solver | [Helper](helper-contracts.md) (chosen per call) | propose a transition | touch funds; make anyone but its own caller lose |
| oracle | [Fetcher](../protocol/oracle/README.md) (chosen per pool) | report the two prices its pool trades at | reach any other pool |
| liquidity | [Vault](../liquidity/vault.md) | hold the LP class like anyone; govern its own book | use any pool permission; breach a gate |
| composition | [Zapper](helper-contracts.md), aggregators, routers | route and convert input tokens | bypass any pool gate |

No admin, pause, or upgrade key touches pool funds anywhere in the system. The only permissioned roles sit outside pool funds: the token metadata descriptor setter, view-only by construction, and the fee receiver's setter and collector, which govern protocol fees already flushed out of the pools.

### Reentrancy

`transition` and `sync` are guarded with transient-storage reentrancy locks; initialization follows checks-effects-interactions. View and quote functions do not check the lock themselves, so a contract that reads pool state from inside its own callback path (an ERC-1155 receiver, a flash-loan callback, a routing aggregator) should call `ensureStateIntegrity()` first; it reverts while the pool is mid-transition. The Vault's state-changing entry points carry the same lock, apart from its pokes, which only forward to the pool's guarded `sync`.
