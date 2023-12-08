# Leverage

Leverage is one of the most useful tools in finance, especially in derivatives trading. When used wisely, leverage can enhance profitability, accelerate wealth growth, maximize opportunities, and optimize capital allocation. However, it is a double-edged sword that is incredibly hard to control. This is mainly because leverage traders are exposed to “liquidation risks”.

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt="" width="563"><figcaption><p>Linear Leverage</p></figcaption></figure>

Compared to holding $10 of ETH, borrowing $20 and opening a $30 Long 3x position will have 3 times more exposure to the ETH price but also to the liquidation risk. As the ETH price decreases, the balance decreases 3 times faster toward $0 while the loan is still $20, increasing the debt-to-balance rate to infinity, which is when the liquidation happens.

For example, Alice has $10. She uses the $10 as collateral to open an ETH Long 10x position in a perpetual exchange. Now, she has $100 worth of ETH after borrowing $90 (leverage) from the exchange. Now, 2 cases can possibly happen:

* Case 1: ETH price decreased by 5%. Alice's position will now be worth $95. Her balance is now worth $5 ($95  - $90). However, you can realize that the leverage of Alice’s position has now changed to 19x ($95 / $5). This leverage is much higher than the 10x she expected, which will continue to rise to infinity as her balance drops to $0. The infinity leverage exposes the lender to systematic risks and is the real reason Alice's position gets liquidated.
* Case 2: ETH price increased by 5%. Alice's position will now be worth $105. Her balance is now worth $15 ($105 - $90). The leverage of Alice’s position has now changed to 7x ($105 / $15). This leverage is lower than the 10x leverage she expected, resulting in a lower capital efficiency for Alice and a lower interest income for the lender (i.e., perpetual exchange). Alice should increase her leverage (risk level) when the market gets risk-on, not reversely.

The problem with liquidation will be solved easily if leverage positions can be rebalanced, i.e., closed and reopened, constantly and consistently. On the downside, rebalancing a position will help leverage traders reduce their risk level/losses and constantly doing so will ultimately make liquidation impossible to happen. Constantly rebalancing a leverage position also benefits the upside because traders can effectively increase their leverage level (risk exposure) as the market gets more bullish and risk-on.

<figure><img src="../.gitbook/assets/image (2) (1).png" alt="" width="563"><figcaption><p>Compounding Leverage</p></figcaption></figure>

It is evident that “rebalancing” is an incredibly powerful tool when combined with leverage, which gives leverage everything it needs: no liquidation, more potential upside, and less downside.

This technique is called “compound”.\
