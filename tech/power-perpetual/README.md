---
description: The First Derivable
---

# Power Perpetual

Power Perpetuals (PP) are the DDL’s derivatives that track the target value of $${B\over Q}^P$$, where B/Q is the rate of any token pair that can be fetched from on-chain oracles, and P is any integer number.

<figure><img src="https://lh3.googleusercontent.com/0fYMpSK8jEToF6GX2tF8I4dTDm0BSHjk6kIe_xWS3nXNFmo1AShob0vfX3zQPlIVjL-wF7nfYjI-kjKGtVMDM7Npr6oCBodwClBnUuVDkPGgkPH5LnCdN7FWmsxi4dkTGVZsV82fufQ5DondNj3u2n6Up-KG5L7IWFd8o8cQnuNaGp0cXwIhl7eX0taS5A" alt=""><figcaption><p>Power Perpetual</p></figcaption></figure>

The Power Perpetual allows token holders to leverage their exposure to both directions of the asset price movement. With positive P, token holders take the LONG position with a leverage rate approximately of P times, and negative P represents the SHORT position.

![](https://lh6.googleusercontent.com/G0zhbLEod7v5dEgclCkyqmVEPjD1kjDR42yb48fDLCLEH97EVfATO7PTx4VsHG6K5m-m3YXW1UHenVcimz1iXwxdoZlA4q\_MYZZ0eYDlUnhY1Lvc0ZqVNCXfI6FbJION7oltNu2Bo-4nolFnSN8GvYHRzBIHDC4goKJ\_7NtjKTYnFl7hbrYhqocH2j3Scw)

Power Perpetual token requires no position nor orderbook, which poses no liquidation risk since $${B\over Q}^P$$ can never go to zero as long as the B/Q is not zero. This advantage does come with some prices: Exposure Fee and Deleveraging Cap.
