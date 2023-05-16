# Pool Creation

Perpetual pools for any token pair can be created using the Derivable application front-end or interact directly with the Derivable smart contract. Each Derivable pool has the following configs:

Reserve Token: the settlement token of the pool.

Price Oracle: the index price of the derivatives (currently, only Uniswap V3 is supported):

* Pool Address: Uniswap V3 pool address.
* Window: the number of seconds to consult the TWAP price. The longer the window, the more resistance to market price manipulation but with more price spread potential in volatile markets.
* Quote Token Index (0 or 1): the index of the quote token of the Uniswap pool.

Mark Price: the square rooted value of the Mark price; it should be as close to the market price as possible and is recommended to be shared with multiple pools of the same Price Oracle.

K: twice the power leverage of the 2 derivative tokens of the pool.

a: initial coefficient of the LONG side.

b: initial coefficient of the SHORT side.

LP Interest Rate: the daily compound interest rate that the LONG and SHORT sides have to pay to the LP.

Init Time: should be as recent as possible and can be the block timestamp of the pool creation transaction.

Min Expiration: LONG and SHORT tokens will be locked for at least this expiration time after minting.

Discount Rate: locktime interest rate discount. (See [expiration.md](../protocol/expiration.md "mention"))
