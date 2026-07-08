---
description: A better IDO system.
---

# Initial Future Offering

{% file src="../.gitbook/assets/Derivable Launchpad IFO (1).pdf" %}

IDO was created as an improvement over ICO to provide liquidity for the issuing token as soon as it is offered to the public. However, it has been swamped by bad actors who constantly attempt to scam uninformed investors.

<figure><img src="../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

Initial Future Offering (IFO) is an upgraded version of the Initial DEX Offering (IDO) process, where the token is also made available on a perpetual DEX for users to go long and short.

<figure><img src="../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

A typical IFO process is as follows:

1. The token developer provides the initial liquidity to a Spot DEX AMM, preferably Uniswap V3, and receives the LP token.
2. Instead of stopping there, the developer also stands up the perpetual market: a Derion pool on the new pair's oracle, with liquidity deposited into the [Vault](../vault/README.md) serving it, for which the developer receives VaultLP shares.
3. The VaultLP can now be either burnt or locked, depending on the tokenomic design chosen by the token developer.

The result is a (much) better IDO system that filters out the bad actors and attracts market confidence.

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
