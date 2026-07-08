# Helper & Periphery

The pool verifies; it does not solve. Everything that computes a trade lives in untrusted periphery that anyone can replace or improve, and the core is designed to remain correct against any malfunctioning or malicious periphery contract.

### Helper

The solver a transactor names in each [transition](../protocol/state-transition.md). Given the pool's snapshot and the caller's intent, `solve()` sizes every leg against the same curve math the pool enforces and returns a complete `Proposal` — the open, close, and flip logic, including the close binary-search, all live here. The pool re-prices and re-verifies everything the Helper returns, so a malicious Helper can only produce a worse fill (or a revert) for its own caller.

### View

A read-only quoting contract that mirrors the pool's state evaluation — including both [funding](../protocol/funding-rate.md) charges — so off-chain estimates track the chain exactly between pokes.

### Zapper

Lets users open and close positions paying or receiving **any ERC-20 or native ETH** in one transaction, routing the non-reserve leg through a DEX aggregator of the caller's choice. It deliberately maintains no aggregator allow-list; safety for everyone else rests on its own invariants: no standing approvals to abuse (input arrives via Permit2 signature or `msg.value`), aggregator allowances granted for the exact in-flight amount and reset to zero in the same call, close proceeds pinned to the position owner, and reentrancy locks throughout. Signed open orders bind the entire order — pool, side, recipient, minimums, aggregator call — into the Permit2 witness, making them relay-safe and front-run-immune.

### TokenDescriptor

View-only metadata for the shared ERC-1155 (see [Derivative Tokens](derivative-tokens.md)). Swappable by the descriptor setter — the protocol's single permissioned role, which cannot touch funds.

### FeeReceiver

Collects and manages the protocol's cut of the [funding flush](../protocol/funding-rate.md).
