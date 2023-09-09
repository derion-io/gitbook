# Pool Creation

<mark style="color:yellow;">(Under Construction)</mark>

As Derivable is an open-market protocol, anyone can create their own liquidity pool for any asset. From the main interface, change to the “Create” tab.

<figure><img src="https://lh3.googleusercontent.com/NdySfq3hH0zZ0Qp5GPiGm0ZoDwwiIuiYA1hN5cgQq6is19enIMc146qTMO8btlBdCJ0OpS2TXvi88-SlWWCVk-Jfe5zAL7ZTSwVMSuEsriUek3XNOpqWxzc2HK0J6mzNW2j55TS5wfOwd6ngXvyKnWU" alt=""><figcaption></figcaption></figure>

Each Derivable pool has the following configs:

* Reserve Token: the settlement token of the pool.
* Price Oracle: the index price of the derivatives (currently, only Uniswap V3 is supported):
  * Pool Address: Uniswap V3 pool address.
  * Window: the number of seconds to consult the TWAP price. The longer the window, the more resistance to market price manipulation but with more price spread potential in volatile markets.
  * Quote Token Index (0 or 1): the index of the quote token of the Uniswap pool.
* Mark Price: the square rooted value of the Mark price; it should be as close to the market price as possible and is recommended to be shared with multiple pools of the same Price Oracle.
* Leverage: the compounding leverage of the 2 derivative tokens of the pool.
* Daily Funding Rate
* Maturity
* Premium Rate
* Opening Rate
