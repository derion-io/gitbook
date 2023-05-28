# Perpetual Leverage

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption><p>Financial Leverage</p></figcaption></figure>

Alice has $10.

On Monday, if she buys ETH, then on Tuesday, after the ETH price increases by 5%, she will profit $0.5.

On Monday, if she opens a Long x10 position in a perpetual exchange. Under the hood, the exchange lends her $90 and buys $100 worth of ETH.

On Tuesday, the ETH price increased by 5%, and Alice's position will now be worth $105. Her balance is now worth $105-$90 = $15; if she closes the position now, she will profit $5, which is 10 times more than $0.05. This is why her position is said to have a leverage of 10.

This leverage, however, is not compounded as the current balance of Alice is not taken into calculation.

On Tuesday, Alice's balance is $15, but her loan is still $90, effectively generating the leverage of only $105/$15 = 7 times. This leverage is lower than the x10 leverage she expects, resulting in lower capital efficiency for Alice and lower interest income from the loaner (i.e., exchange).

In another timeline, on Tuesday, the ETH decreased by 5%, and Alice's position will now be worth $95. Her balance is now worth $95-$90 = $5, but her loan is still $90, effectively generating the leverage of $95/$5 = 19 times. This leverage is much higher than the x10 leverage she promised and continues to rise to infinity as the ETH price drops. The infinity leverage exposes the loaner to systematic risks and is the real reason Alice's position gets liquidated. This erratic leverage is the consequence of non-compounding financial mathematics inherited from the legacy of paper's future contracts.



