# State Transition

A pool state is represented by a 3-tuple ⟨R,α,β⟩, and it can be changed by user transactions.

![](<../.gitbook/assets/image (33).png>)![](<../.gitbook/assets/image (17).png>)

State transitions can alter the curves, affecting the effective leverage of derivative tokens, but they cannot change the value of existing positions in the pool. Effective leverage can only be reduced ([deleveraged](broken-reference)) and only for one side that is nearly out of reserve.

The ability of liquidity providers to deleverage one side of the pool is equivalent to an Uniswap LP’s ability to increase pool slippage by withdrawing a large portion of the liquidity. However, the cost of manipulation is high, and the resulting damage is almost negligible.
