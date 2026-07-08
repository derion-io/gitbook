# Price Oracle

### Uniswap v3

Uniswap v3 is the built-in oracle: any token pair on the network is ready for a Derion pool without extra infrastructure. The pool's `ORACLE` config packs the pair address, the quote-token index, and the TWAP window; a longer window resists price manipulation better, at the cost of more spread in fast markets.

<figure><img src="../.gitbook/assets/image (45).png" alt=""><figcaption><p>Uniswap V3 TWAP Oracle</p></figcaption></figure>

### Custom Fetchers

A pool may instead name a custom `FETCHER` contract as its oracle logic, adapting any price source — Chainlink feeds, other DEXs, or any exotic index. Because a pool's config is immutable and hashed into its address, a hostile or manipulable fetcher can only ever harm its own pool: its risk never leaks into the rest of the protocol.

### The double price system

The oracle reports two prices — TWAP and spot — and every trade is filled at the **bound less favorable to the transactor**. Derivative pricing is prediction, and any latency between the two prices is exploitable; always charging the adverse bound turns that latency into a spread paid by the trader instead of an arbitrage against the pool's liquidity.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Double Price System</p></figcaption></figure>

Which bound is adverse depends on the trade's direction, and the direction is only fully known after the [Helper](../design/helper-contracts.md) has solved the trade. Selection therefore runs in two stages:

1. **Pre-pick.** Before the Helper runs, the only directional signal is the transactor's declared slippage floors, so the pool folds the *signs* of $$\Delta A_{\min}$$ and $$\Delta B_{\min}$$ into a single net exposure and optimistically picks the bound adverse to it: net long → MAX (long is dear), net short or neutral → MIN.
2. **Re-check.** After the trade is solved, the pool recomputes the net exposure from the *realized* deltas,

$$
E \;\propto\; \frac{\Delta A}{s_A} - \frac{\Delta B}{s_B}
$$

and requires the bound actually used to be adverse to it — whenever the two oracle prices diverged. When TWAP and spot agree, the pick is irrelevant.

A wrong declaration can never steal. Fishing for the favorable bound — declaring short to grab MIN on a trade that ends up net long — trips the re-check and reverts the caller's own transaction. Mis-declaring into a *more* adverse bound only gives the caller a worse fill. In every branch, the only party a wrong declaration can hurt is the caller — which is what makes a user-supplied hint safe to trust.

{% hint style="info" %}
When the oracle bounds have diverged, all-zero slippage floors default the pre-pick to MIN — so a net-long trade must declare its direction or revert. The slippage floors double as a direction declaration whenever the oracle spread is live.
{% endhint %}
