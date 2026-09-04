# Helper & Periphery

The pool verifies; it does not solve. Everything that computes a trade lives in untrusted periphery that anyone can replace or improve, and the core is designed to remain correct against any malfunctioning or malicious periphery contract.

### Helper

The solver a transactor names in each [transition](../protocol/state-transition.md). Given the pool's snapshot at both oracle bases and the caller's intent, `solve()` sizes every leg and returns a complete `Proposal`: the two post-trade coefficients and the four signed deltas. Every solve is closed-form. A mint leg is priced at its class's dear bound and a burn leg at its cheap bound, which is exactly what clears the pool's charge at both bases; the coefficients are then aimed at whichever basis' floors bind, the protected side exactly at its pin and the traded side at what the settlement leaves after that pin and the LP class's fee-raised floor.

The reference Helper's payload is `abi.encode(sideIn, sideOut, amount)` for opens, closes, flips, and LP deposits and withdrawals, and a sentinel-prefixed tuple for rotations (open one side, close the other, one net reserve leg). See the [Pool API](../contracts/api.md) for the exact formats. The pool re-prices and re-verifies everything the Helper returns, so a malicious Helper can only produce a worse fill, or a revert, for its own caller.

### View

A read-only quoting contract that mirrors the pool's evaluation, including both [funding](../protocol/funding-rate.md) charges and the pending protocol cut, so off-chain estimates track the chain exactly between pokes. It reports the three class reserves and supplies, both oracle prices, and the pool's metadata. `EOAProbe` is a companion for dry-running a whole transaction via state override: installed at the caller's address, it forwards the call, captures gas and return data, and accepts the ETH and ERC-1155 transfers a real account would.

### Factory

Deploys pools as MetaProxies over the shared logic and, optionally, seeds them in the same transaction. `deployWithStrategy` additionally writes a Vault strategy that extends an existing one with the new pool, so a market can be deployed, initialized, and proposed for Vault funding in one call ([Strategy Governance](../liquidity/governance.md)). Every deployment emits a creation log carrying the encoded config and the pool address, which is how indexers discover pools.

### Zapper

Lets users open and close positions paying or receiving **any ERC-20 or native ETH** in one transaction, routing the non-reserve leg through a DEX aggregator of the caller's choice. It deliberately maintains no aggregator allow-list; safety for everyone else rests on its own invariants: no standing approvals to abuse (input arrives via Permit2 signature or `msg.value`), aggregator allowances granted for the exact in-flight amount and reset to zero in the same call, close proceeds pinned to the position owner, and reentrancy locks throughout. Signed open orders bind the entire order (pool, side, recipient, minimums, aggregator call) into the Permit2 witness, making them relay-safe and front-run-immune.

The Zapper compiles against the current pool interface but its test suite is not part of the green build; treat it as pending re-verification before use.

### TokenDescriptor

View-only metadata for the shared ERC-1155 (see [Derivative Tokens](derivative-tokens.md)). Swappable by the descriptor setter, the protocol's single permissioned role, which cannot touch funds.

### FeeReceiver

Collects the protocol's cut of [funding](../protocol/funding-rate.md), flushed to it by pool pokes. It affects only the protocol fee recipient, never pool users.
