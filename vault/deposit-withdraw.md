# Deposit & Withdraw

Anyone can supply capital to the Vault. `deposit(amount)` pulls reserve tokens in and mints **VaultLP** — the Vault's own ERC-20 share — pro-rata to NAV; `withdraw(shares)` burns VaultLP and pays reserve back out.

### NAV

The Vault's net asset value is its idle reserve plus every active position marked at its pool's close quote:

```
NAV = idle TOKEN_R + Σ getAmountOut(position → reserve) over served pools
```

Positions are marked by what they would actually fetch on close — through the same curve and oracle every trader is priced on — not by a model.

### The directional spread

Share pricing is deliberately asymmetric:

* **Deposits** mark NAV at the **higher** close bound — the depositor is minted fewer shares, protecting existing holders.
* **Withdrawals** mark NAV at the **lower** close bound — the withdrawer is paid less, protecting the remaining holders.

The transactor eats the spread in both directions, and each pool's TWAP anchors the bounds against single-block manipulation. This is the same adverse-selection principle the pools themselves use, applied at the share boundary.

### Withdrawing past idle

When a withdrawal exceeds the idle reserve, the Vault auto-closes positions to raise the difference. Positions are closed whole, so a large withdrawal can pull more depth than strictly needed — the overshoot simply stays idle for the next deployment or withdrawal.

### What depositors carry

VaultLP is not a fixed-income claim. Depositors earn the [funding](../protocol/funding-rate.md) flushed from every served pool, and carry:

* the **counterparty P&L** of the Vault's book — the impermanent loss and gain of standing opposite the traders' net position (quantified in [LP Economics](lp-economics.md));
* **policy risk** — the active [strategy](governance.md) decides which pools get funded and how aggressively.

All of it is priced at the share boundary, fully isolated from trader safety: no trader payout ever depends on the Vault's solvency.
