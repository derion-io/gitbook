# Funding Rate

There are two components to the Funding Rate: the Interest Rate and the Premium Rate.

Unlike conventional perpetual exchanges, where the funding rate is manually charged and updated periodically, Derion's funding rate is autonomous and continuously applied on every block of the underlying smart contract platform.

<figure><img src="../.gitbook/assets/fee.gif" alt=""><figcaption><p>Interest and Protocol Fee</p></figcaption></figure>

## Interest Rate

The Interest Rate is continuously charged from both the Long and Short sides to the LP side. Each pool is configured with a base interest rate I (expressed as an annual compounding rate).

A total of 1/5 of the interest tokens paid to LP is collected as the protocol fee.

#### Effective Leverage

Due to Derion's power perpetual curves, positions can be in two regimes:

* **Power regime** (leveraged): Position has full leverage $K$
* **Asymptotic regime** (deleveraged): Position leverage decreases as price moves further

The **effective leverage** of each side ($k\_L$ for Long, $k\_S$ for Short) is calculated based on the current price position on the curve:

<p align="center"><span class="math">k_L = \min(K, k(x))</span></p>

<p align="center"><span class="math">k_S = \min(K, k(x))</span></p>

Where `k(x)` is the instantaneous leverage at current price `x`. In the power regime, $$k = K$$. In the asymptotic regime, $$k < K$$ and decreases toward 0 as the position deleverages.

#### Compounding Interest Rate

The actual interest rate paid by each side scales with their effective leverage:

**Long Interest Rate:**

<p align="center"><span class="math">i_L = I \times \frac{k_L}{K}</span></p>

**Short Interest Rate:**

<p align="center"><span class="math">i_S = I \times \frac{k_S}{K}</span></p>

This means:

* At full leverage ($$k=K$$): Pay the full base interest rate
* When deleveraged ($$k<K$$): Pay **less** interest, proportional to effective leverage

This makes economic sense: deleveraged positions have less market exposure (delta), so they pay less for that reduced exposure to LP.

**LP Interest Rate (received):**

<p align="center"><span class="math">i_{LP} = \frac{(r_A + r_B) \times I}{r_C}</span></p>

A total of 1/5 of the interest paid to LP is collected as the protocol fee.

## Premium Rate

The Premium is paid by the larger side directly to the smaller side of Long and Short, offering them the chance of negative funding rates as an incentive to balance the market. **Note: LP does not receive premium — it flows only between Long and Short.**

Each pool is configured with a maximum premium rate P (expressed as an annual compounding rate).

#### Compounding Premium Rate

The premium rate scales with the market imbalance between Long and Short:

**Long Premium Rate:**

<p align="center"><span class="math">p_L = P \times \frac{(r_A - r_B) \times (r_A + r_B)}{R \times r_A}</span></p>

**Short Premium Rate:**

<p align="center"><span class="math">p_S = P \times \frac{(r_B - r_A) \times (r_A + r_B)}{R \times r_B}</span></p>

Where:

* $$r_A$$: Long reserve
* $$r_B$$: Short reserve
* $$R$$: Total pool reserve
* Positive rate means paying premium
* Negative rate means receiving premium

This design ensures:

* When $$r_A > r_B$$: Long pays premium, Short receives premium
* When $$r_B > r_A$$: Short pays premium, Long receives premium
* When $$r_A = r_B$$: No premium is exchanged

The premium rate is **not affected by deleveraging** — it depends only on the reserve imbalance regardless of the curve regime.

### Total Funding Rate

The total funding rate for each side is the sum of interest and premium:

<p align="center"><span class="math">f_L = i_L + p_L</span></p>

<p align="center"><span class="math">f_S = i_S + p_S</span></p>

<p align="center"><span class="math">f_{LP} = -i_{LP}</span></p>

(LP receives interest, hence negative funding)

### Protocol Fee

The protocol fee is calculated from the increase in LP reserves (from interest) and transferred to the fee receiver:

<p align="center"><span class="math">fee = \frac{r_C' - r_C}{5}</span></p>

Where $$r_C' = R - r_A' - r_B'$$ is the new LP reserve after interest and premium are applied, and $$r_C$$ is the original LP reserve before any fees.

This fee produces an actual token transfer and reduces the total pool reserve:

<p align="center"><span class="math">R' = R - fee</span></p>
