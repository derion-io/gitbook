# Value Invariant

Whatever shape a transition takes, it has to pass one final check before it commits. Writing $$s_A, s_B$$ for the side supplies and primes for post-trade values, the pool requires the per-share value of **both** sides to be non-decreasing:

$$
\frac{r_A'}{s_A'} \;\ge\; \frac{r_A}{s_A} - \varepsilon
\qquad\text{and}\qquad
\frac{r_B'}{s_B'} \;\ge\; \frac{r_B}{s_B} - \varepsilon
$$

up to a per-share rounding dust $$\varepsilon$$. Every unit of reserve belongs to exactly one of the two share classes ($$r_A + r_B = R$$), so these two inequalities are the entire safety statement. The transactor's own mint or burn is already counted in the post supplies $$s'$$, so the check permits only *them* to lose — their spread and fees, which their own slippage floors bound separately. No one but the transactor can lose value in a transition.

The check is direction-agnostic — it holds for any combination of $$\langle \Delta A, \Delta B, \Delta R \rangle$$ — which is what makes multi-direction trades safe with the same check the simple open/close path uses. Funding is not a counter-example: it is applied before the pre-trade snapshot, so the "before" inventory is already post-funding. The arithmetic runs at 512-bit intermediate precision, and the denominators are always positive because the pool's initialization mints unburnable dead shares on both sides.

### Consolidated invariants

1. $$\text{balance} \ge R$$ — the funding outbox $$\text{balance} - R$$ is non-negative and never enters the engine.
2. $$r_A + r_B = R$$ — the engine reserve is fully partitioned between the two sides.
3. $$0 \le r_A, r_B < R$$ — the asymptotic cap: neither side can drain the engine, so there is no liquidation.
4. $$R_1 = R - \Delta R$$ on every transition — the engine moves in lockstep with the tokens.
5. Per-share value is non-decreasing for both sides, up to $$\varepsilon$$ — no one but the transactor loses value.
6. Funding moves value one way only: from the dominant trader side to the liquidity provider, applied before the trade snapshot.
7. Whenever the oracle prices diverge, the selected price is adverse to the transactor's realized net exposure.
8. A pool's reserve is reachable only through its own transitions — there is no cross-pool custody.
