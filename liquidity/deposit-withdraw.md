# Deposit & Withdraw

Anyone can supply capital to the Vault. `deposit(amount, payment)` pulls reserve tokens in and mints **VaultLP**, the Vault's own ERC-20 share, pro-rata to NAV; `withdraw(shares, helper)` burns VaultLP and pays reserve back out.

### Payment

A deposit takes the same `Payment` struct as a pool ([Payments](../design/payments.md)): native ETH is wrapped in place when the reserve is WETH, the remainder is pulled by Permit2 signature or by a plain `transferFrom`, and on the signature path the recipient must be the signer. With a Permit2 signature a user never approves the Vault itself. The first deposit locks a small number of dead shares as an inflation guard.

### NAV

The Vault's net asset value is its idle reserve plus every LP-class position marked at its pool's close quote:

$$
\text{NAV} = \text{idle} + \sum_{\text{pools}} \text{getAmountOut}(C \to R,\ \text{position})
$$

Positions are marked by what they would fetch on close, through the same curve and oracle every trader is priced on, not by a model. Funding is settled on every active pool (`pokeAll`) before NAV is read, so it is never stale, and the promotion rule of [Strategy Governance](governance.md) keeps the Vault's positions inside the active set, so the walk over active pools covers the whole book.

### The directional spread

Share pricing is deliberately asymmetric:

* **Deposits** mark NAV at the **higher** close bound. The depositor is minted fewer shares, protecting existing holders.
* **Withdrawals** mark NAV at the **lower** close bound. The withdrawer is paid less, protecting the remaining holders.

The transactor eats the spread in both directions, and each pool's TWAP anchors the bounds against single-block manipulation. This is the adverse-selection principle the pools use, applied at the share boundary. A manipulated spot cannot mint cheap shares or drain more than the adverse-bound NAV allows.

### Withdrawing past idle

When a withdrawal exceeds the idle reserve, the Vault closes positions to raise the difference, pool by pool, each close sized to the remaining shortfall (padded slightly for quote drift) and capped at the position. It pays exactly the marked amount or reverts; it never overpays. The `helper` argument is the caller's solver for those closes and is untrusted: every close is floored at the pool's own adverse-bound quote.

A forced close under a diverged oracle is priced at the LP class's cheap bound, and the withheld gap stays in the pool's residual, most of which is the Vault's own remaining position. What a full exit from a pool leaves behind accrues to that pool's remaining LP supply.

### Locked shares

Shares escrowed as a governance vote cannot be withdrawn until unlocked. They keep their full NAV claim throughout.

### What depositors carry

VaultLP is not a fixed-income claim. Depositors earn the growth of every LP position the Vault holds (the [funding](../protocol/funding-rate.md), [fees](../protocol/opening-fee.md), and divergence spreads of every served pool) and carry:

* the **counterparty P&L** of the Vault's book, a short straddle against the traders' net position in every served pool ([LP Economics](lp-economics.md));
* **policy risk**: the active strategy decides which pools get funded and how aggressively.

All of it is priced at the share boundary and isolated from trader safety. No trader payout ever depends on the Vault's solvency.
