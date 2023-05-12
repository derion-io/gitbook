# Derivative Tokens

All derivatives (LONG and SHORT) and Liquidity tokens are different IDs of a single [ERC-1155](https://eips.ethereum.org/EIPS/eip-1155) token. Token ID is encoded in the following format:

`ID = 0x##a...a`

Where `0xa...a` is its pool address, and `0x##` is the side code of the token in the pool:

* `0x10`: LONG side
* `0x20`: SHORT side
* `0x30`: Liquidity Provider side

Each token ID can only be minted and burnt by its pool.

The ERC-1155 token standard is used with 2 extensions: Timelock and Shadow.

[Timelock](https://github.com/derivable-labs/erc1155-timelock) extension allows the token to be locked for a specific amount of time after its being minted.

[Shadow](https://github.com/derivable-labs/shadow-token) extension allows an ERC-20 token to be deployed for each ID on-demand. This ERC-20 token shares the same balances and allowances as the original ERC-1155's ID but has its own address to be composed with other DeFi applications (e.g. Uniswap's pool).&#x20;
