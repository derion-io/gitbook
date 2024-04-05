# Power Perpetuals

Real-time compounding, i.e., constantly closing and re-opening positions, is impractical even with today's computing power. Any gap between re-investing circles will definitely generate systematic risks. Fortunately, there's already a practical, equivalent solution that has been put to the test by Opyn Squeeth: [Power Perpetuals](https://www.paradigm.xyz/2021/08/power-perpetuals)

{% embed url="https://www.paradigm.xyz/2021/08/power-perpetuals" %}
Power Perptuals
{% endembed %}

Power Perpetuals are financial derivatives that track index values in the form of xⁿ. After conducting mathematical calculations (refer to our [Whitepaper](../whitepaper.md)), we found out that Power Perpetuals can replicate exactly the result of compounding a leverage position.

With the benefits that compounding offers, we believe that using Power Perpetuals to replicate compounding in our protocol will provide a better leverage and risk system for the users & trading pools, further allowing Derion to work more seamlessly and effectively as a Perpetuals AMM DEX.\
