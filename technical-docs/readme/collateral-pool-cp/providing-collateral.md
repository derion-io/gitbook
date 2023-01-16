# Providing Collateral

Collateral pools must have their liquidity initialized before the derivative tokens can be converted.

## **Pool Creation**

A pool is created for the first time an amount of C ($$\Delta^0_C$$) is added. The pool is initialized with:

* $$\Delta^0_{CP} = S^0_{CP} = R^0_{BC} = \Delta^0_C$$

$$\Delta^0_{CP}$$ is the amount of CP token received by the pool creator, (100% share of the pool).

$$S^0_{CP}$$ is the supply of the CP token of this pool.

$$R^0_{BC}$$ is the initial reserve of the Buffer Collateral pool.

## Add Collateral

Locking additional $$\Delta^1_C$$ collateral to the pool will mint $$\Delta^1_{CP}$$ of CP token.

* $$\Delta^1_{CP} = S^0_{CP} \times \Delta^1_C \div R^0_{BC}$$
* $$S^1_{CP} = S^0_{CP} + \Delta^1_{CP}$$
* $$R^1_{BC} = R^0_{BC} + \Delta^1_C$$

## Remove Collateral

Burning $$\Delta^2_{CP}$$ amount of CP token allows users to unlock $$\Delta^2_C$$ from the pool.

* $$\Delta^2_C = R^1_{BC} \times \Delta^2_{CP} \div S^1_{CP}$$
* $$S^2_{CP} = S^1_{CP} - \Delta^2_{CP}$$
* $$R^2_{BC} = R^1_{BC} - \Delta^2_C$$

