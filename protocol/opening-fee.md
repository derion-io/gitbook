# Opening Fee

A pool may charge a fee on opening Long and Short positions through its `OPEN_RATE` config: the fraction of the gross payment that becomes position, so a rate of 1 (Q128) is no fee, and so is a rate of 0, the off switch. Closes pay nothing, and neither do deposits into or withdrawals from the LP class. Providing liquidity pays no leverage fee.

### Where it lands

The pool computes the fee floor itself from the realized mint legs, valued pro-rata at each oracle basis, and enforces it twice ([Value Gates](value-invariant.md)): the transactor's charge must include it, and the LP class's per-share floor is raised by it. The first makes the fee binding even though the solver is transactor-supplied; the second directs it to the LP class rather than to the incumbents of the side being minted. The fee is not transferred anywhere. It arrives as growth of $$r_C / s_C$$.

There is no waiver for any address. The pool knows no provider, and the [Vault](../liquidity/vault.md) pays no fee on its deposits only because LP-class legs are fee-free for everyone.

### When to charge one

Zero is the fair default. With the interest half-life set at break-even for the asset, every holder already pays per unit of time exactly what their position costs the LP class, and an entry fee would be a second charge on the same thing, falling hardest on whoever holds shortest.

The fee has one job the continuous charge cannot do: flow around a *predictable* move. A straddle bought minutes before a scheduled event and closed minutes after pays almost no interest and takes the whole convexity of the move. The fee is a toll on that flow, and its bound is

$$
f \;\ge\; \cosh(k\, j_{\text{event}}) - 1
$$

for an event with a typical absolute log move $$j_{\text{event}}$$. For crypto majors and macro prints at $$k \le 4$$ the bound is under 1%. For single stocks with earnings it is the binding constraint on leverage: a name that moves 8% on earnings cannot carry more than ×3 with a fee traders will pay. When the bound exceeds what traders will accept, the answer is a lower leverage, not a higher fee. [Pool Parameters](../guide/pool-parameters.md) has the full sizing.
