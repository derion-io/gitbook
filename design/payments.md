# Payments

A transition's reserve leg can be paid in two ways, selected by the `Payment` struct. The [Vault](../liquidity/deposit-withdraw.md)'s deposit takes the same struct with the same rules.

```solidity
struct Payment {
    address recipient;  // where minted tokens / paid-out reserve go (required)
    address owner;      // Permit2 signer; on the signature path MUST equal recipient
    uint256 nonce;      // Permit2 nonce
    uint256 deadline;   // Permit2 deadline
    bytes signature;    // empty → transferFrom(msg.sender); non-empty → Permit2
}
```

### Direct allowance

With an empty `signature`, the pool pulls the reserve with `transferFrom(msg.sender)`. This path suits contracts and routers holding a direct allowance; the recipient can be freely chosen.

### Permit2

With a signature, the reserve is pulled by [Permit2](https://github.com/Uniswap/permit2) signature transfer, so no standing approval to the pool is ever needed. The pool requires `recipient == owner` on this path, so a relayer submitting a signed order cannot redirect the output away from the signer. This is what makes signed orders relay-safe.

### Native ETH

Pools reserved in WETH accept and deliver native ETH: `msg.value` is wrapped on the way in, and setting `unwrapETH` in the transition unwraps the payout on the way out. Only the canonical wrapped-native token may be settled this way; a reserve token that merely accepts ETH is rejected, so a payment leg can never be marked paid without the pool receiving reserve.

### Oracle update fees

On a pool whose fetcher is a pull oracle ([Price Oracle](../protocol/oracle/README.md)), `msg.value` may also carry the fee for a caller-supplied price update. The pool forwards native value to the fetcher only when `oracleData` is non-empty; the fetcher spends at most the update's cost and refunds the rest, and the pool then wraps what remains as reserve (on a WETH pool) or returns it to the caller. A bundled oracle fee is never double-counted as reserve, and an over-estimate comes back.

### Closing by transfer

A position of any class can be closed without calling the pool at all: transfer the ERC-1155 into the pool with an encoded `Param` as the data, and the transfer callback runs the transition with the recipient pinned to the sender, pays the reserve out, and refunds any part of the position the close did not consume.
