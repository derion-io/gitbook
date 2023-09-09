# Derivative Tokens

All derivatives (LONG and SHORT) and Liquidity tokens are different IDs of a single [ERC-1155](https://eips.ethereum.org/EIPS/eip-1155) token. Token ID is encoded in the following format:

`ID = 0x##a...a`

Where `0xa...a` is its pool address, and `0x##` is the side code of the token in the pool:

* `0x10`: LONG side
* `0x20`: SHORT side
* `0x30`: Liquidity Provider side

Each token ID can only be minted and burnt by its pool.

The ERC-1155 token standard is deployed with 2 extensions: Timelock and Shadow.

[Maturity](https://github.com/derivable-labs/erc1155-maturity): an implementation of [ERC-1155](https://eips.ethereum.org/EIPS/eip-1155) that tracks a maturity time after an amount of token is minted. (See [maturity.md](../protocol/maturity.md "mention") for more details)

[Shadow](https://github.com/derivable-labs/shadow-token): an extension of [ERC-1155](https://eips.ethereum.org/EIPS/eip-1155) that allows any of its IDs to deploy a Shadow [ERC-20](https://eips.ethereum.org/EIPS/eip-20) token with its contract address for DeFi composability. Each Shadow contract can be deployed by anyone, anytime, and it shares the same balance and transfer behavior with the original token ID after being deployed.
