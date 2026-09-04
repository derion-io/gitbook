---
description: The shared, governed anchor holder of the LP class
---

# Liquidity Vault

The Vault is one ownerless, permissionless balance sheet serving every pool that shares its reserve token. It holds liquidity in two forms:

* **Idle liquidity**: reserve tokens sitting on the Vault, waiting to be deployed.
* **Active liquidity**: real [LP-class](lp-class.md) positions the Vault opens through each pool's ordinary `transition()` path, one position per pool.

The Vault has **no pool permissions**. A pool knows no LP by name, so anything the Vault can do any other LP-class holder can do; the Vault is distinguished only by its policy layer. It decides *which* pools to fund by a vote of its own depositors ([Strategy Governance](governance.md)), *how much* by a target that keeps each pool's dominant side at full leverage, and *how fast* by a per-pool ramp on its own idle ([Depth Provision](depth-provision.md)). Its shares are priced at NAV with a directional spread ([Deposit & Withdraw](deposit-withdraw.md)).

Every Vault trade passes the same gates as everyone else's: its own slippage floors, the transactor's charge, and the three per-share floors at both oracle bases. It pays the adverse bound like anyone. That is the price of holding no privileges, and it is what makes depth provision manipulation-resistant without any pool-side machinery: an attacker steering the Vault's fills pays the spread every trader pays and risks only the Vault depositors' capital, never trader collateral.

### Trader safety does not depend on the Vault

Payouts come from each pool's own reserve, capped below $$R$$ by the [curve](../protocol/pricing.md), with no dependence on Vault solvency and no top-up path. If the Vault is absent or idle, the pool remains a pure trader-versus-trader engine and nothing freezes. A missed rebalance costs quality of service, in that the dominant side's leverage sags below target, never solvency. This asymmetry is what lets the Vault be permissionless, rate-limited, and optional, where a linear-payoff design would need its equivalent to be fast, incentivized, and privileged.

### Capital and vote are the same share

Depositing reserve mints `VaultLP`, the Vault's own ERC-20, pro-rata to NAV; withdrawing burns it. The same share, escrowed behind a candidate strategy, is the governance vote, weighted by how long it has been locked. The capital at risk is what chooses how it is deployed, and the escrow keeps its full NAV claim.

Depositors bear the counterparty P&L of the Vault's book, the short-straddle exposure of every LP position it holds, plus the policy risk of the active strategy. All of it is priced at the share boundary and isolated from trader safety.

### Why a Vault at all

Anyone can hold the LP class directly. The Vault adds what a single holder cannot easily provide: capital that follows a rule instead of a person, so pools receive depth in proportion to their imbalance without anyone watching; governance that can add and drop pools as a group; a ramp that limits how fast a bad pool can absorb shared capital; and one fungible share over a diversified book.
