# State Transition

All trades go through a single state-changing entry point: `transition()`. The pool does not solve anything itself — it snapshots its state, lets an untrusted **Helper** chosen by the caller propose the complete transition, then re-prices, verifies, and settles it. A dishonest Helper can only hurt its own caller.

### The delta basis

Every quantity is signed, from the transactor's perspective: positive means they receive, negative means they give.

The transactor's request carries only their protection and their intent:

$$
\langle\, \Delta A_{\min},\ \Delta B_{\min},\ \Delta R_{\min} \,\rangle
$$

— floors on the change of their Long balance, their Short balance, and the reserve they receive — together with the Helper's address and an opaque intent payload for it.

The Helper answers with the proposed transition, every part of it untrusted:

$$
\langle\, \alpha_1,\ \Delta A,\ \Delta B,\ \Delta R \,\rangle
$$

— the Long coefficient after the trade and the three signed deltas. The post-trade engine reserve is never proposed; it is fixed by settlement,

$$
R_1 = R - \Delta R
$$

since the engine moves in lockstep with the tokens. (The exact call signatures live in the [API reference](../contracts/api.md).)

### The flow

1. Snapshot the side supplies $$s_A, s_B$$.
2. Select the price bound from the signs of the declared floors ([Price Oracle](oracle.md)).
3. Accrue [funding](funding-rate.md) at that price.
4. The Helper solves the trade and returns $$\langle \alpha_1, \Delta A, \Delta B, \Delta R \rangle$$ — untrusted.
5. Re-evaluate the post-trade reserves at the pool's own selected price: $$r_{A,1} = \rho(\alpha_1 x^k,\, R_1)$$ and $$r_{B,1} = R_1 - r_{A,1}$$.
6. Re-check the price bound against the realized net exposure ([Price Oracle](oracle.md)).
7. Check the transactor's own slippage floors.
8. Verify the [value invariant](value-invariant.md).
9. Commit $$\langle R_1, \alpha_1 \rangle$$ and settle the deltas.

The Helper is a convenience, never a trust assumption: step 5 re-evaluates the curve at the pool's own selected price, so a lying proposal can never be priced against faked reserves, and any other dishonesty is caught by one of the three gates (6, 7, 8). All the trade math a pool used to carry internally — open, close, flip, sizing — now lives in the [Helper](../design/helper-contracts.md).

### Slippage and settlement

The floors

$$
\Delta X \ge \Delta X_{\min} \qquad \text{for } X \in \{A,\, B,\, R\}
$$

bound the transactor's *own* loss — their spread and fees. Everyone else is protected by the [value invariant](value-invariant.md), not by the floors. A receiver floors what they get ($$\Delta X_{\min} > 0$$); a giver caps what they give ($$\Delta X_{\min} < 0$$); the minimum integer value stands for $$-\infty$$, no floor at all.

Settlement is by sign: each side with $$\Delta > 0$$ is minted to the recipient and each side with $$\Delta < 0$$ is burned from the payer; reserve owed ($$\Delta R < 0$$) is pulled in ([direct allowance or Permit2](../design/payments.md)) and reserve received ($$\Delta R > 0$$) is paid out, optionally unwrapped to native ETH. A position can also be closed by simply transferring it into the pool — the transfer callback burns it and pays out the reserve.

### Multi-direction trades

Because one price bound is picked per transition, the legal multi-leg combinations are exactly the **rotations** that agree on a single bound: receive A / give B, or give A / receive B — flips, close-one-open-the-other, trim one side while growing the other. Opening or closing *both* sides in one transition would want opposite bounds and is rejected by the exposure re-check. The single-coefficient engine collapses all legs into one signed scalar, so there is no straddle case and no special conflict handling.

### Rounding tolerance

The pool cannot tell an exact forward solve from a slightly overshooting inverse solve — the method is hidden inside the Helper — so every transition allows a bounded rounding dust (on the order of 1 wei per share at normal prices) on the invariant check. This is the acknowledged cost of keeping the solver outside the pool.
