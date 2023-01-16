# Price Tolerance Ratio

Price Tolerance Ratio (PTR) is the rate of price movement that the pool is targeted to absorb. The C reserve (RC) collateralizes RDC on both sides of the price movement:

* $$R_C ≥ max(R^+_{DC}, R^-_{DC})$$

Where:

* __$$R^+_{DC} = R^{T\times PTR}_{DC}$$ ($$R_{DC}$$ when the price is PTR times increased from the current TWAP)
* $$R^-_{DC} = R^{T\div PTR}_{DC}$$ ($$R_{DC}$$ when the price is PTR times decreased from the current TWAP)

PTR only restricts Collateral Providers from removing their liquidity, the rest of the pool operations are always available as the Derivative users should understand the risk and benefits of their actions.

\
