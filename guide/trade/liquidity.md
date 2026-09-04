# Providing Liquidity

Liquidity on Derion is a token you hold, not a role you sign up for. Every pool has an [LP class](../../liquidity/lp-class.md): the reserve its Long and Short pay-offs leave over, issued as the pool's third position token. There are two ways to hold it.

### Directly, in one pool

Deposit the pool's reserve token and receive its LP position. The pool prices your deposit at the LP class's adverse oracle bound, exactly as it prices a Long or Short, and charges no opening fee. Withdraw any time by closing the position back to reserve. You pick the market and the size, and your exposure is to that one pool's book.

### Through the Vault

Deposit reserve tokens into the shared [Vault](../../liquidity/vault.md) and receive **VaultLP**, an ERC-20 share of the Vault's balance sheet, priced at NAV. The Vault deploys capital as LP positions across the pools its active strategy covers, keeping each market's dominant side at full leverage, and its depositors vote on which pools those are. Withdraw any time; the Vault closes positions if its idle reserve is short. Details in [Deposit & Withdraw](../../liquidity/deposit-withdraw.md).

### What you earn and carry

Either way, the income is the same and arrives the same way: as growth in the reserve behind each LP share, with nothing to claim.

* **Income**: the [funding](../../protocol/funding-rate.md) traders pay (interest from both sides, premium from the crowded side), the [opening fees](../../protocol/opening-fee.md), and the spread between the oracle's two prices on trades executed while they diverge, all net of the protocol's cut of funding.
* **Risk**: the LP class is the counterparty. Its value is highest at the price center and falls as the price moves either way, so you are short volatility against the traders' net position. Through the Vault you also carry the policy risk of the active strategy. [LP Economics](../../liquidity/lp-economics.md) works out when the position is profitable.

Deposits and withdrawals in both paths are priced with a protective spread, so entering or leaving cannot dilute the other holders. If you are the only provider in a pool and exit entirely while the oracle's two prices disagree, the withheld spread stays behind with the pool's dead shares; see [The LP Class](../../liquidity/lp-class.md).
