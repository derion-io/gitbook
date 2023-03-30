# Impermanent Loss

LIQUIDITY token is often less exposed to the index value compared to LONG and SHORT. Like any other liquidity tokens, they are also exposed to Impermanent Loss (IL) as the index value is changed by market forces.

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

To compensate for the IL risks, LP earns the price spread and funding rate from LONG and SHORT.

Unlike position-based perpetual exchanges, where the funding rate is used for LONG and SHORT to compensate each other, the funding rate in Derivable is used for LONG and SHORT to compensate the LP. The compensation between LONG and SHORT naturally happens when the curve deleverages itself.
