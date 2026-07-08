# Value Invariant

Whatever shape a transition takes, it has to pass one final check before it commits:

```solidity
function _verifyInv(Inv memory pre, Inv memory post, uint256 tol) internal pure {
    if (mulDiv(post.rA, pre.sA, post.sA) + tol < pre.rA) revert INVARIANT_A;
    if (mulDiv(post.rB, pre.sB, post.sB) + tol < pre.rB) revert INVARIANT_B;
}
```

Every unit of reserve belongs to exactly one of two share classes ($$r_A + r_B = R$$), so the invariant is simply: **each class's per-share value must not decrease**, up to the rounding tolerance. The transactor's own mint or burn is already counted in the post supplies, so the check permits only *them* to lose — their spread and fees, which their own slippage floors bound separately. No one but the transactor can lose value in a transition.

The check is direction-agnostic — it works for any combination of `(dA, dB, dR)` — which is what makes multi-direction trades safe with the same code the simple open/close path uses. Funding is not a counter-example: it is applied before the pre-trade snapshot, so the "before" inventory is already post-funding. The division is 512-bit, and the denominators are always positive because the pool's initialization mints unburnable dead shares on both sides.

### Consolidated invariants

1. `balance ≥ R` — the funding outbox `balance − R` is non-negative and never enters the engine.
2. `rA + rB = R` — the engine reserve is fully partitioned between the two sides.
3. `0 ≤ rA < R` and `0 ≤ rB < R` — the asymptotic cap: neither side can drain the engine, so there is no liquidation.
4. `R₁ = R − dR` on every transition — the engine moves in lockstep with the tokens.
5. Per-share value is non-decreasing for both sides, up to the rounding tolerance — no one but the transactor loses value.
6. Funding moves value one way only: from the dominant trader side to the liquidity provider, applied before the trade snapshot.
7. Whenever the oracle prices diverge, the selected price is adverse to the transactor's realized net exposure.
8. A pool's reserve is reachable only through its own transitions — there is no cross-pool custody.
