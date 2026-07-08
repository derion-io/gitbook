# Opening Fee

A pool may charge a fee on opening positions through its `OPEN_RATE` config (a rate of 1 means no fee). The fee is not transferred anywhere: it stays in the pool, so it accrues to the existing holders of the side being minted. Its purpose is friction — making short-cycle open/close loops around oracle latency unprofitable for MEV bots in markets that need the extra protection.

The fee is **waived when the pool mints to its `PROVIDER`**: depth provision by the [Vault](../vault/README.md) grows the reserve by exactly the amount paid in, with no skim on the service. Keying the waiver on the recipient is safe because an opening fee only ever benefits a side's existing holders, and minting to the provider can only benefit the provider itself.

Pools built primarily for Vault-provided depth may simply set `OPEN_RATE` to no fee and rely on the [adverse price selection](oracle.md) for manipulation resistance.
