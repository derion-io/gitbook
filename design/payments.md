# Payments

A transition's reserve leg can be paid in two ways, selected by the `Payment` struct:

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

With a signature, the reserve is pulled by [Permit2](https://github.com/Uniswap/permit2) signature transfer — no standing approval to the pool is ever needed. The pool binds the EIP-712 witness to the `recipient` and requires `recipient == owner`, so a relayer submitting a signed order cannot redirect the output away from the signer. This is what makes signed orders relay-safe.

### Native ETH

Pools reserved in WETH accept and deliver native ETH: `msg.value` is wrapped on the way in, and setting `unwrapETH` in the transition unwraps the payout on the way out.

### Closing by transfer

A position can be closed without calling the pool at all: transfer the ERC-1155 into the pool, and the transfer callback burns it and pays the reserve out to the owner.
