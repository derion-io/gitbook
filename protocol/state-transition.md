# State Transition

All trades go through a single state-changing entry point: `transition()`. The pool does not solve anything itself — it snapshots its state, lets an untrusted **Helper** chosen by the caller propose the complete transition, then re-prices, verifies, and settles it. A dishonest Helper can only hurt its own caller.

### The delta basis

Every quantity is signed, from the transactor's perspective: positive means they receive, negative means they give.

```solidity
struct Param {          // the transactor's request
    int256 dAMin;       // floor on the change of their LONG balance
    int256 dBMin;       // floor on the change of their SHORT balance
    int256 dRMin;       // floor on the reserve they receive
    address helper;     // untrusted solver
    bytes payload;      // trade intent (sides, magnitude) for the Helper
    bool unwrapETH;     // deliver received reserve as native ETH
}

struct Proposal {       // the Helper's answer — every field untrusted
    uint256 a;          // LONG coefficient after the trade
    int256 dA;          // δ transactor's LONG
    int256 dB;          // δ transactor's SHORT
    int256 dR;          // reserve received (pool δR = −dR)
}
```

The post-trade engine reserve is never proposed — it is fixed by settlement, `R₁ = R − dR`, since the engine moves in lockstep with the tokens. The Helper proposes only the coefficient and the signed deltas.

### The flow

```
1. snapshot the side supplies sA, sB
2. select the price bound from the declared floors         (Price Oracle)
3. accrue funding at that price                            (Funding)
4. Helper.solve(...) → Proposal                            untrusted
5. re-evaluate rA₁, rB₁ from (a, R − dR) at the real price
6. re-check the price bound against the realized exposure  (Price Oracle)
7. check the transactor's own slippage floors
8. verify the value invariant                              (Value Invariant)
9. commit (R, a) and settle the deltas
```

The Helper is a convenience, never a trust assumption: step 5 re-evaluates the curve at the pool's own selected price, so a lying proposal can never be priced against faked reserves, and any other dishonesty is caught by one of the three gates (6, 7, 8). All the trade math a pool used to carry internally — open, close, flip, sizing — now lives in the [Helper](../design/helper-contracts.md).

### Slippage and settlement

`dX ≥ dXMin` for each of A, B, R bounds the transactor's *own* loss — their spread and fees. Everyone else is protected by the [value invariant](value-invariant.md), not by the floors. A receiver floors what they get (`dXMin > 0`); a giver caps what they give (`dXMin < 0`); `type(int256).min` means no floor.

Settlement is by sign: each side with `δ > 0` is minted to the recipient and each side with `δ < 0` is burned from the payer; reserve owed is pulled in ([direct allowance or Permit2](../design/payments.md)) and reserve received is paid out, optionally unwrapped to native ETH. A position can also be closed by simply transferring it into the pool — the transfer callback burns it and pays out the reserve.

### Multi-direction trades

Because one price bound is picked per transition, the legal multi-leg combinations are exactly the **rotations** that agree on a single bound: receive A / give B, or give A / receive B — flips, close-one-open-the-other, trim one side while growing the other. Opening or closing *both* sides in one transition would want opposite bounds and is rejected by the exposure re-check. The single-coefficient engine collapses all legs into one signed scalar, so there is no straddle case and no special conflict handling.

### Rounding tolerance

The pool cannot tell an exact forward solve from a slightly overshooting inverse solve — the method is hidden inside the Helper — so every transition allows a bounded rounding dust (on the order of 1 wei per share at normal prices) on the invariant check. This is the acknowledged cost of keeping the solver outside the pool.
