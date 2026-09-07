---
description: Providing liquidity to a pool directly
---

# The LP Class

The LP class is the pool's third share class: the residual $$r_C = R - r_A - r_B$$ of the two [curves](../protocol/pricing.md), issued as ERC-1155 id `0x30` of the pool. It is minted and burned through `transition()` like the sides, carries dead shares from initialization like the sides, and is worth $$r_C / s_C$$ per share.

### Deposit and withdraw

A deposit pays reserve in and mints $$\text{amount} \cdot s_C / r_C$$ shares at the class's **dear** bound, the higher of its residuals at the two oracle bases, hence the fewest shares. A withdrawal burns shares and credits $$\text{shares} \cdot r_C / s_C$$ at the **cheap** bound. Both directions clear the class's per-share floor by construction and pay the boundary spread like every class; the TWAP anchors it against single-block manipulation. Neither pays an [opening fee](../protocol/opening-fee.md).

The residual is not monotone in price (it peaks at the center), so unlike the sides its dear and cheap bounds are found by comparing the two realized residuals rather than by picking the higher or lower price. The pool's `getAmountOut` quote does the same.

On a deposit the Helper aims both side coefficients so that each side's reserve is preserved at both bases and the residual takes the whole reserve leg; on a withdrawal the residual gives it. Below the inflection this is exact. Under a diverged oracle with a saturated side, a deposit may be resized down to what the realized residual backs at both bases, bounded by the depositor's own slippage floor. A withdrawal has no mint to resize; if the squeezed residual cannot hold its floor, the call reverts cleanly.

### What a share is

Three things at once.

**A counterparty position.** As the price moves, the winning side grows faster than the losing side shrinks, and the difference comes out of $$r_C$$. For a fixed state $$(R, \alpha, \beta)$$ the residual is a pure function of price: it loses on any move away from the center and recovers fully on a return. It is a short straddle on $$p^k$$.

**Depth.** The inflection of $$\rho$$ is at $$R/2$$, and the residual is part of $$R$$, so a funded LP class keeps both sides on their full-leverage branch longer. It pads both curves at once, so there is no side to choose.

**The recipient of every revenue stream.** [Funding](../protocol/funding-rate.md) decays both sides into the residual at every touch; the opening fee is forced onto it by its fee-raised floor; the divergence gap on a diverged trade lands on it at the non-binding basis. All of it arrives as per-share growth, with no flush and nothing to claim.

### Anyone can enter, and why that is safe

A class that collects fees and funding and that anyone may join invites the question of whether a late entrant can capture what standing holders earned. Four bounds answer it.

* **Funding cannot be back-run.** A deposit runs the funding accrual first, so a just-in-time depositor mints at the post-funding price and captures none of what accrued before it arrived.
* **A round trip pays the spread.** Entry is at the dear bound and exit at the cheap one, both TWAP-anchored, so a deposit and withdrawal taken while the oracle is diverged returns less than it put in. A divergence cannot be flipped into a round-trip profit in either direction.
* **Manufacturing a divergence does not pay.** The gap a diverged trade lands on $$r_C$$ is paid by that trade's own transactor. An attacker who forces the divergence and trades pays the gap; one who only round-trips pays the spread. There is no victim to create.
* **Dilution is pro-rata and principal-safe.** A depositor sandwiching someone else's fee'd or diverged trade captures at most its capital-weighted share of that one accrual. A standing holder's principal is never touched; only income is shared, exactly as in any pro-rata pool. This is ordinary just-in-time dilution, the accepted residual property.

### Full exits under divergence

A holder closing their entire LP position while the oracle is diverged is credited at the cheap bound and leaves the withheld gap to the remaining LP supply. For the last real holder that remainder is the dead shares, and the value is burned. It is the exiting holder's own cost, the same spread every class pays at the boundary, but worth knowing before exiting a pool you alone provide for.

### Tooling

The LP class is a token. It moves, it can be held by contracts, and it can be closed by transferring it into the pool. Position events cover LP legs with a mark-to-curve value like the sides, so indexers see liquidity flows without special handling beyond recognizing the `0x30` id. See [Derivative Tokens](../design/derivative-tokens.md).
