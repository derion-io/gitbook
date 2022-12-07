---
description: Manage Wallet Exposure and Value
---

# Exposure

{% embed url="https://derivable.io/exposure" %}
Derivable Exposure Interface
{% endembed %}

![](<../.gitbook/assets/image (3).png>)

Information:

* WBNB/BUSD: the tracking price
* 303.46: current market price
* \+4.47%: 24h change
* 44.50 BUSD: net value of user wallet (a.k.a position)
* \+30.63%: 24h change for user wallet net value
* Long x6.8: Long/Short direction and active leverage

Actions:

* Add: add more Cake-LP to your wallet exposure
* Remove: remove Cake-LP from your wallet exposure
* Slide: current wallet exposure and desired wallet exposure
* Swaps: (auto-generated) underlying actions need to execute the change. (MUST approved token for correct calculation)
* Deleverage: deleverage the pool before executing transactions
* Execute: send the transaction to your wallet (MetaMask, Trust, etc.)
* Reset: clear the change