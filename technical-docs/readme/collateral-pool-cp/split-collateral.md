# Split-Collateral

The C reserve is split into 2 portions: Derivative Collateral and Buffer Collateral. The DC always covers the same market value of the Derivative token supply minted by this pool. The BC takes the rest of the C reserve to absorb all the market volatility of the entire supply of D, providing the over-collateralized property for the Derivative asset.

<figure><img src="https://lh5.googleusercontent.com/IyyOzv2oYKfFIw5wYiYIF6LnPwfj_mZt4ZeItqiA609ThqC5vACKWsYDHN49ERm6LsxR5pbJBB79VbOSXquJqYNhCmo-HVCrW5EddmPxCtUwZ11snRculONZsRrKd0hu24xzrszU0wusIVfgN1YV0PJ_C3YJbN_flKpdZiGJsaOrJJF0j3hN9msM2ExZFA" alt=""><figcaption><p>Spli-Collateral Pool</p></figcaption></figure>

In a multiple tokens pool design, multiple Derivative tokens Di tracking multiple Target Value Vi is minted from a single collateral Pool:

* $$S_D = \sum{S_{Di}}$$
* $$R_C$$ is the total reserve of C in the pool
* $$S_{Di}$$ is the supply of $$D_i$$ minted by this pool
* $$E_{DiC}$$ is the current exchange rate from $$D_i$$ to C

We have the reserve of C for Derivative Collateral:

* $$R_{DCi} = S_{Di}\times E_{DiC}$$
* $$R_{DC} = \sum{R_{DCi}}$$

And the reserve of C for Counter Collateral:

* $$R_{BC} = R_C - R_{DC}$$
