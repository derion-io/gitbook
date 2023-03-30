---
description: Swap one token to another.
---

# Swap

{% embed url="https://derivable.io/#/swap" %}
Swap Interface
{% endembed %}

![](<../.gitbook/assets/image (1) (2) (1).png>)

Tokens:

* Cake-LP is the LP token of PancakeSwap for the WBNB+BUSD pair. Cake-LP can be acquired from [pancakeswap.finance](https://pancakeswap.finance/add/BNB/0xe9e7CEA3DedcA5984780Bafc599bD69ADd087D56). But you can always swap directly from BNB, WBNB, or BUSD in the Swap interface.
* DDL-CP is the Collateral Provider token of the pool. Swapping from WBNB/BUSD/Cake-LP is providing collateral to the pool. Swapping from DDL-CP to Cake-LP is withdrawing collateral from the pool.
* $$BNB^x$$: Derivative Tokens for BNB powered of x.

Restriction:

* Cannot swap any token to WBNB or BUSD
* DDL-CP can only swap to CAKE-LP (removeLiquidity)
* Only WBNB, BUSD, or Cake-LP can swap to DDL-CP. (Not from BNB^x)

Conversion Fee (BETA):

* 0.3% fee is taken when swapping from DDL-CP to Cake-LP (removeLiquidity)
* No fee is taken for all other swaps

ERC20 and ERC1155 token approval for Derivable’s Router is required.
