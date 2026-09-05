# AMM-LP IL Hedge

A liquidity provider in a constant-function AMM (a Uniswap v2 pair or a v3 range) is short convexity. The position is worth less than simply holding the two tokens whenever the price moves away from where the liquidity was added, in either direction, and the shortfall grows with the size of the move. That shortfall is impermanent loss, and its shape is the shape of a short straddle.

A Derion pool offers the other side of that trade. Holding equal reserve in the Long and the Short of one pool is a long straddle on $$p^k$$: below the inflection the pair is worth $$r\,(x^k + x^{-k})$$, smallest at the entry price and larger after any move ([Three Share Classes](../protocol/engine.md#a-matched-pair-is-a-straddle)). Sized against the AMM position, the pair's gain on a move offsets the LP's impermanent loss, and the [funding](../protocol/funding-rate.md) the pair pays is the price of the hedge, as an option premium would be.

The match is not exact. Impermanent loss is one fixed curve in the price ratio, while the straddle's curvature is set by $$k$$ and its pay-off flattens on the asymptotic branch, so the hedge ratio drifts with the price and needs occasional rebalancing. What it gives in return is a hedge that lives on-chain, never expires, and cannot be liquidated.

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>
