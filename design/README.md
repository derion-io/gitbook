# Technical Design

Derion comprises a set of well-isolated, immutable smart contracts, designed for deployment on any EVM-compatible chain.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### One logic contract, one proxy per pool

A single **Pool Logic** contract carries all the code — the curve, the funding, the transition verification. Each pool is a separate **ERC-3448 MetaProxy** deployed with CREATE2, delegatecalling that shared logic. The pool's immutable `Config` is baked into the proxy bytecode itself and forwarded on every call, so reading it costs no storage access; only the mutable state (the reserve, the coefficient, the funding clock) lives in storage.

**Config is the address.** The CREATE2 address hashes the init code, which embeds the encoded config — so a configuration maps deterministically to exactly one pool address. The same config cannot be deployed twice, and anyone can derive a pool's address from its config alone. Pool creation is permissionless and automatically de-duplicated.

### Why per-pool proxies, not a singleton

Every pool runs the same logic, so isolation is not about code bugs — it is about custody. Each proxy holds only its own reserve token, which bounds any exploit to a single pool's reserve and forces an attacker to re-run it pool by pool. Decisively, a **hostile configuration** — a malicious fetcher, a manipulable oracle, or an exotic fee-on-transfer reserve token — can harm only the pool that chose it. A singleton would commingle every pool's reserve in one pot.

### Trust tiers

| Tier | Contract | Can | Cannot |
| ---- | -------- | --- | ------ |
| law + custody | Pool Logic + proxy (per pool) | hold funds; enforce the curve and the value invariant; mint/burn its sides | — |
| solver | [Helper](helper-contracts.md) (chosen per call) | propose a transition | touch funds; make anyone but its own caller lose |
| liquidity | [Vault](../vault/README.md) (`PROVIDER`) | trade like anyone; collect funding as payee | use any pool permission; breach an invariant |
| composition | [Zapper](helper-contracts.md), aggregators | route and convert input tokens | bypass any pool gate |

No admin, pause, or upgrade key touches pool funds anywhere in the system. The single permissioned role in the protocol is the token metadata descriptor setter — view-only by construction.

### Reentrancy

`transition` and `sync` are guarded with transient-storage reentrancy locks; initialization follows checks-effects-interactions. View functions additionally reject read-only reentrancy, so quoting contracts cannot be fooled mid-transition.
