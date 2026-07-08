# Providing Liquidity

Liquidity on Derion is provided through the [Vault](../../vault/README.md), not into individual pools. Depositing reserve tokens mints **VaultLP** — an ERC-20 share of the Vault's balance sheet — and can be undone at any time by withdrawing.

What your deposit does: the Vault deploys capital as real Long/Short positions across the pools its active strategy covers, keeping each market's dominant side at full leverage. In exchange, every served pool streams its [funding](../../protocol/funding-rate.md) — interest and premium paid by traders — to the Vault.

What you earn and carry:

* **Income**: the funding flushed from every served pool, net of the protocol fee.
* **Risk**: the counterparty P\&L of the Vault's book — you are effectively short volatility against the traders' net position — plus the policy risk of the active strategy. See [LP Economics](../../vault/lp-economics.md) for when this position is profitable.

Deposits and withdrawals are priced at NAV with a protective directional spread, so entering or leaving cannot dilute the other depositors — details in [Deposit & Withdraw](../../vault/deposit-withdraw.md).
