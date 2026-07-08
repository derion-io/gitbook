---
description: The external shared liquidity provider
---

# Liquidity Vault

The Vault is Derion's liquidity layer: one ownerless, permissionless balance sheet serving every pool that shares its reserve token and pins it as the pool's `PROVIDER`.

It holds liquidity in two forms:

* **Idle liquidity** — reserve tokens sitting on the Vault, waiting to be deployed.
* **Active liquidity** — real Long/Short positions the Vault opens through each pool's ordinary `transition()` path. Providing depth means opening a position on the minor side: reserve flows into the pool, $$R$$ grows, and the dominant side walks back toward its inflection point — back onto full leverage.

The Vault has **no pool permissions**. It passes the same four gates as every trader — the [adverse price bound](../protocol/oracle.md), its own slippage floors, the [value invariant](../protocol/value-invariant.md), and the structural reserve check — and pays the adverse oracle bound like anyone. That is the price of holding no privileges, and it is what makes depth provision manipulation-resistant without any special pool-side machinery: an attacker steering the Vault's fills pays the same spread every trader pays, and risks only the Vault depositors' capital, never trader collateral. The one concession the pool makes is waiving its [opening fee](../protocol/opening-fee.md) on mints to the `PROVIDER`.

In return, the Vault is the pool's [funding](../protocol/funding-rate.md) payee: interest and premium accrue in-pool on every touch and flush to the Vault on every poke.

### Trader safety does not depend on the Vault

Payouts come from each pool's own reserve, capped below $$R$$ by the [curve](../protocol/pricing.md), with no dependence on Vault solvency and no top-up path. If the Vault is absent or idle, the pool remains a pure trader-vs-trader engine; nothing freezes. A missed rebalance costs quality of service — the dominant side's effective leverage sags below target — never solvency. This asymmetry is exactly what lets the Vault be permissionless, rate-limited, and optional, where a linear-payoff design would need its equivalent machinery to be fast, incentivized, and privileged.

### Two decoupled surfaces

* **Capital** — deposit reserve tokens to mint `VaultLP`, the Vault's own ERC-20 share, pro-rata to NAV; withdrawing burns it. Depositors bear the counterparty P&L of the Vault's book plus its policy risk, priced at the share boundary and isolated from trader safety. See [Deposit & Withdraw](deposit-withdraw.md).
* **Governance** — which pools the Vault funds, and how fast, is the active Strategy, voted by locking LP tokens. Votes carry no `VaultLP` claim; economic claim and vote weight are separate by design. See [Strategy Governance](governance.md).

Directional risk nets across pools, but deployed depth does not — it is reserve physically committed per pool, which is exactly what keeps every trader prefunded. Capital reuse comes from reallocating *idle* liquidity, not from shrinking commitments.
