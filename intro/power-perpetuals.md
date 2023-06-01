# Power Perpetuals

In Alice's situation ([leverage.md](leverage.md "mention")), theoretically, if she keeps closing and re-opening her position constantly, day and night, without being charged any additional fee, everyone would be happy because:

* Alice's position keeps updating her effective leverage to a constant of 10 times regarding to her current balance.
* Alice's loan is always 9 times of the balance, so her balance can never run out, and her position will never need to be liquidated.
* Lenders don't have to worry about under-collateralized loans and liquidation risks.

Constantly closing and re-opening positions is impractical even with today's computing power. Any gap between re-investing circles will definitely generate systematic risks. Fortunately, there's already a practical solution that has been put to the test by Opyn Squeeth: [Power Perpetuals](https://www.paradigm.xyz/2021/08/power-perpetuals)

{% embed url="https://www.paradigm.xyz/2021/08/power-perpetuals" %}
Power Perptuals
{% endembed %}

Power Perpetuals are financial derivatives that track index values in the form of $$ETH^P$$; they are exactly what Alice will have if she constantly closes and re-opens her positions (i.e., compounding positions).
