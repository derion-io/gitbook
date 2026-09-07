# State Transition

All trades go through a single state-changing entry point: `transition()`. The pool does not solve anything itself. It snapshots its state, lets an untrusted **Helper** chosen by the caller propose the complete transition, then re-prices, verifies at both oracle bases, and settles. A dishonest Helper can only hurt its own caller.

### The delta basis

Every quantity is signed from the transactor's perspective: positive means they receive, negative means they give.

The transactor's request carries their protection and their intent:

$$
\langle\, \Delta A_{\min},\ \Delta B_{\min},\ \Delta C_{\min},\ \Delta R_{\min} \,\rangle
$$

floors on the change of their Long, Short, and LP balances and on the reserve they receive, together with the Helper's address, an opaque intent payload for it, a flag to receive reserve as native ETH, and an optional oracle update for the pool's fetcher.

The Helper answers with the proposed transition, every part of it untrusted:

$$
\langle\, \alpha_1,\ \beta_1,\ \Delta A,\ \Delta B,\ \Delta C,\ \Delta R \,\rangle
$$

the two coefficients after the trade and the four signed deltas. The post-trade engine reserve is never proposed. It is fixed by settlement,

$$
R_1 = R - \Delta R
$$

so the engine moves exactly with the tokens and nothing is routed anywhere else. (Call signatures are in the [Pool API](../contracts/api.md).)

### The flow

1. Snapshot the three supplies $$s_A, s_B, s_C$$.
2. Fetch both oracle bases, TWAP and spot, forwarding any caller-supplied oracle data ([Price Oracle](oracle/README.md)).
3. Accrue [funding](funding-rate.md) at the TWAP. This yields the pre-trade side reserves at the TWAP basis. If the oracle diverged, the pool recovers each moved side's coefficient from those reserves, without persisting it, and evaluates the spot-basis reserves from the recovered pair. The LP residual at each basis is $$R - r_A - r_B$$.
4. The Helper solves the trade against this snapshot and returns $$\langle \alpha_1, \beta_1, \Delta A, \Delta B, \Delta C, \Delta R \rangle$$.
5. Check the transactor's own slippage floors.
6. Fix $$R_1 = R - \Delta R$$.
7. At each basis, re-evaluate the post-trade curves $$r_{A,1} = \rho(\alpha_1 x^k, R_1)$$ and $$r_{B,1} = \rho(\beta_1 x^{-k}, R_1)$$, take the residual $$r_{C,1}$$, and run the [value gates](value-invariant.md): the charge and the three per-share floors. The spot pass is skipped when the oracle did not diverge.
8. Commit $$\langle R_1, \alpha_1, \beta_1 \rangle$$ and settle the deltas.

The Helper is a convenience, never a trust assumption. Step 7 re-evaluates the curves from the proposed coefficients at the pool's own prices, so a proposal cannot be priced against faked reserves, and any other dishonesty fails one of the gates. Because pricing is at a point rather than along a curve, every leg is pro-rata at the snapshot reserves and the solve is closed-form for every operation.

### Slippage and settlement

The floors $$\Delta X \ge \Delta X_{\min}$$ for $$X \in \{A, B, C, R\}$$ bound the transactor's own loss, their spread and fees. Everyone else is protected by the value gates. A receiver floors what they get, a giver caps what they give, and the minimum integer stands for no floor at all.

Settlement is by sign. Each class with $$\Delta > 0$$ is minted to the recipient and each with $$\Delta < 0$$ is burned from the payer; reserve owed is pulled in ([direct allowance or Permit2](../design/payments.md)) and reserve received is paid out, optionally as native ETH. A position of any class can also be closed by transferring it into the pool: the transfer callback runs the transition and pays out, refunding whatever part of the position the close did not consume.

### Operations

The reference Helper solves five single-direction operations and one compound one:

* **Open** Long or Short: pay reserve, mint at the class's dear bound, the higher of its two reserves, hence the fewest shares. The [opening fee](opening-fee.md) is added to the gross payment.
* **Close** Long or Short: burn shares, receive their value at the class's cheap bound.
* **Flip** Long to Short or back: a burn at the cheap bound feeding a fee'd mint at the other side's dear bound, with no reserve leg.
* **Deposit** into the LP class: pay reserve, mint at the LP class's dear bound. No fee.
* **Withdraw** from the LP class: burn shares, receive at the cheap bound.
* **Rotate**: open one side and close the other in one transition, with a single net reserve leg.

Per-leg worst-case pricing (mint dear, burn cheap) is exactly what clears the charge at both bases. For the sides, the dear bound is the higher price for Long and the lower for Short. The LP class's reserve is not monotone in price, so its dear and cheap bounds are found by comparing the two realized residuals directly.

Having sized the legs, the Helper aims the two coefficients at whichever basis' floors bind. For a side operation one side is protected and aimed exactly at its per-share pin: the side being burned, or on a plain open the side not being minted. The other side, the minted side on an open or a flip and the untouched complement on a close, takes what the settlement leaves after that pin and the LP class's fee-raised floor, shaved by its own quantum. Subtracting the raised floor is what leaves the fee on the LP class. For an LP deposit or withdrawal both side coefficients are aimed to preserve their reserves at both bases, and the residual takes or gives the whole reserve leg.

Under a diverged oracle with a saturated side, the reference Helper may resize a mint, a side open or an LP deposit, down to what the capped reserves back at both bases; the transactor absorbs that reduction, bounded by their own slippage floor. A withdrawal has no mint to resize and reverts cleanly instead.

### Rounding tolerance

The pool cannot tell an exact forward solve from an inverse solve that overshoots by one unit, so every gate allows a bounded dust per class: each side's own coefficient quantum, capped at a $$2^{-32}$$ fraction of that side's pre-trade reserve, plus a few wei; the LP class, which absorbs both sides' recovery dust, gets the sum. The cap matters on pools whose price has drifted far from the mark: uncapped, the quantum would grow until the gates stopped binding. Past the cap a solver must land the curve exactly or overpay, a liveness cost on far-drifted pools and never a leak. The most any transition can take for free stays below gas cost at any reserve size.

One consequence is a known corner: a deeply saturated side on a high-power pool whose oracle has diverged may be unsolvable within tolerance, and the operation reverts cleanly with no state change. The corner is transient in price relative to the mark and heals as the price returns toward it. Integrators should size down or wait on such pools.
