# Collateral Asset

Collateral asset (C) is locked in the pool the whole time while not involved in the value tracking process, making yield-bearing tokens like [Uniswap LP Tokens](https://blog.instadapp.io/unipool-maker-vaults) the best choice for it. The price of Base and Quote tokens are already fed from Uniswap oracle, so the price of Uniswap LP (LP\_B\_Q) is also available.

A simple Router contract can help acquire the LP\_B\_Q token from B and/or Q before being locked into the pool as the C token. The same process can be done for removing the C token, but users are recommended to keep the LP token instead of converting back to B and/or Q if they are planning to be locked in again.
