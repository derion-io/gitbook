# Trader Incentive

Trader Incentive is a unique program where losing positions will be compensated with DER proportional to the value lost. This means that if a trader predicts the correct market direction, they will receive [rETH](https://rocketpool.net) or [BNBx](https://www.staderlabs.com) as profits. However, if they go in the wrong market direction, they will receive DER as insurance.

21% of the initial supply of DER is locked in the Trader Incentive contract to compensate for losing traders. Please refer to [rewards.md](rewards.md "mention") for the exact calculation.

All positions that apply for Trader Incentive will have their value (in [rETH](https://rocketpool.net) or [BNBx](https://www.staderlabs.com)) tracked upon opening and closing transactions. If the reserve value of the position on closing is less than its value on opening, the user will be compensated with a share amount of DER proportional to the value difference. The more value is lost, the larger the share of DER that will be compensated. The DER Reward Cap determines the total amount of DER distributed to each share of the compensation.
