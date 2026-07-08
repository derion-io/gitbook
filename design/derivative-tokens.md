# Derivative Tokens

All Derion positions are ids of one shared [ERC-1155](https://eips.ethereum.org/EIPS/eip-1155) token contract (with supply tracking), serving every pool. A token id encodes its pool and side:

`ID = 0x##a...a`

where `0xa...a` is the pool address and `0x##` is the side code:

* `0x10`: LONG
* `0x20`: SHORT

(`0x00` and `0x01` are reserved to denote the reserve-token and native-ETH settlement legs, and `0x30` is reserved for LP sides where a pool issues one — used by the [Vault's governance escrow](../vault/governance.md).)

### The open mint/burn rule

Instead of a registry of authorized pools, the token uses one open rule: **any address may mint and burn every id that ends in its own address.** A pool therefore controls exactly its own sides — and only those — with no permission, no allow-list, and nothing to administer. The same rule lets any future contract issue its own 1155 sides on the shared token.

A pool never holds its own tokens: positions transferred into a pool are burned as part of the transfer-and-call close path, paying the reserve out to their owner.

### Metadata

Names, symbols, and display metadata come from a swappable, view-only **descriptor** contract. The descriptor setter is the only permissioned role in the protocol; it can change how positions render in wallets and nothing else.
