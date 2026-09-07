# Value Gates

Whatever shape a transition takes, it must pass one gate set at each oracle basis before it commits. Write $$s_X$$ for the class supplies, primes for post-trade values, and $$\varepsilon_A, \varepsilon_B$$ for the two sides' rounding dust. All reserves are evaluated at the basis under test, TWAP or spot.

### The charge

$$
\Delta R + \Delta A\,\frac{r_A}{s_A} + \Delta B\,\frac{r_B}{s_B} + \Delta C\,\frac{r_C}{s_C} \;\le\; -\,\text{fee}_{\min} + \varepsilon_A + \varepsilon_B
$$

The transactor's whole take across the three classes must cost at least its value at this basis plus the [opening fee](opening-fee.md). Requiring it at both bases is the adverse charge: the dear basis is the strict one. Mint legs round up and burn legs round down, both against the transactor.

### The three floors

$$
r_A'\,\frac{s_A}{s_A'} + \varepsilon_A \;\ge\; r_A
\qquad
r_B'\,\frac{s_B}{s_B'} + \varepsilon_B \;\ge\; r_B
\qquad
r_C'\,\frac{s_C}{s_C'} + \varepsilon_A + \varepsilon_B \;\ge\; r_C + \text{fee}_{\min}
$$

where $$r_A', r_B'$$ are re-evaluated by the pool from the proposed coefficients and $$r_C' = R_1 - r_A' - r_B'$$ is the residual, saturating at zero. Each is a per-share floor written with the post-trade reserve rescaled to the pre-trade supply, so that the dust allowance reads in reserve units. No class's per-share value drops at either basis. The transactor's own mint or burn is already in the post supplies, so the floors let only them lose, and their own slippage floors bound that loss separately.

**The fee-raised LP floor is the fee's routing.** The charge makes the transactor pay $$\text{fee}_{\min}$$; raising the LP class's floor by the same amount is what makes the fee land there rather than wherever the transactor-supplied Helper might aim it. A fee-dodging proposal fails the charge. A proposal that pays the fee but aims it at a side fails the LP floor.

**Why the LP floor is per-share.** The side floors are one-sided: a proposal that gifts the residual to a side's incumbents is invisible to them and to the charge, and only the LP floor catches it. With the LP class tokenized, the same floor also prices the class's own deposits and withdrawals exactly like the sides'. One rule, three classes.

**Solvency falls out.** The residual saturates at zero, so a proposal with $$r_A' + r_B' > R_1$$, equivalently $$\alpha_1\beta_1 > (R_1/2)^2$$, fails the LP floor outright. There is no separate product check.

Funding is not a counter-example to any of this: it is applied before the pre-trade snapshot, so the "before" inventory is already post-funding. The arithmetic is 512-bit, and the denominators are positive because every class carries unburnable dead shares from initialization.

### Consolidated invariants

1. $$\text{balance} \ge R$$, and $$\text{balance} - R$$ is invariant across transitions. Only funding's protocol cut feeds it; only a poke drains it.
2. $$\alpha\beta \le (R/2)^2 \iff r_A + r_B \le R$$ at every price $$\iff r_C \ge 0$$ at any price. Enforced at initialization and preserved by the LP floor.
3. $$0 \le r_A, r_B < R$$: the asymptotic cap. No liquidation.
4. $$R_1 = R - \Delta R$$ exactly on every transition. The engine is structural.
5. Per-share value of A, B, and C is non-decreasing at both bases, C's floor raised by the opening fee, each up to its class's dust. No class but the transactor loses; the fee and the divergence gaps land on the LP class.
6. The charge holds at both bases over all three legs: the transactor pays at least the adverse valuation of their take plus the fee.
7. Funding decays the sides into the residual: the LP class collects, traders pay, and only the protocol cut leaves the engine.
8. Every class's supply has an unburnable dead-share floor.
9. A pool's reserve is reachable only through its own transitions.
