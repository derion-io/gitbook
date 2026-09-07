# Derivative Tokens

All Derion positions are ids of one shared [ERC-1155](https://eips.ethereum.org/EIPS/eip-1155) token contract (with supply tracking), serving every pool. A token id encodes its pool and side:

`ID = 0x##a...a`

where `0xa...a` is the pool address and `0x##` is the side code:

* `0x10`: LONG
* `0x20`: SHORT
* `0x30`: LP, the [LP class](../liquidity/lp-class.md)

`0x00` and `0x01` denote the reserve-token and native-ETH settlement legs and are never minted.

All three classes are ordinary fungible positions: they transfer, they can be held by contracts, and any of them can be closed by transferring it into its pool. An LP position of one pool has nothing to do with `VaultLP`, which is the [Vault](../liquidity/vault.md)'s own ERC-20 over its whole book.

### The open mint/burn rule

Instead of a registry of authorized pools, the token uses one open rule: **any address may mint and burn every id that ends in its own address.** A pool therefore controls exactly its own classes, and only those, with no permission, no allow-list, and nothing to administer. The same rule lets any future contract issue its own ERC-1155 sides on the shared token.

Two consequences for integrators. Do not assume the low 160 bits of an arbitrary id are a Derion pool; to filter for real pools, recompute the MetaProxy address from the pool's `loadConfig()` and compare. And `totalSupply(id)` is only meaningful for ids that resolve to a pool, since anyone can inflate or deflate the supply of ids ending in their own address.

A pool never holds its own tokens: positions transferred into a pool are burned as part of the transfer-and-call close path, paying the reserve out to their owner and refunding any unconsumed part.

### Metadata

Names, symbols, and display metadata come from a swappable, view-only **descriptor** contract. The descriptor setter is permissioned, one of the two administrative roles in the protocol (the other manages the protocol-fee receiver); it can change how positions render in wallets and nothing else.
