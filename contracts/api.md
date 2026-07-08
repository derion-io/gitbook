# API

## transition

All trades — opens, closes, flips, and multi-leg rotations — go through one function on the pool:

```solidity
function transition(
    Param memory param,
    Payment memory payment
) external payable returns (int256 dA, int256 dB, int256 dR, uint256 price);

struct Param {
    int256 dAMin;       // floor on δ(your LONG balance)
    int256 dBMin;       // floor on δ(your SHORT balance)
    int256 dRMin;       // floor on the reserve you receive
    address helper;     // untrusted solver contract
    bytes payload;      // intent, decoded by the helper
    bool unwrapETH;     // deliver received reserve as native ETH
}

struct Payment {
    address recipient;  // where minted tokens / paid-out reserve go (required)
    address owner;      // Permit2 signer; on the signature path MUST equal recipient
    uint256 nonce;      // Permit2 nonce
    uint256 deadline;   // Permit2 deadline
    bytes signature;    // empty → transferFrom(msg.sender); non-empty → Permit2
}
```

Every delta is signed from the caller's perspective: positive = received, negative = given. The returns are the realized deltas and the selected oracle price.

The three minimums are the caller's slippage protection *and* their direction declaration for [price selection](../protocol/oracle.md): a receiver floors what it gets (`> 0`), a giver caps what it gives (`< 0`), and `type(int256).min` means no floor. See [State Transition](../protocol/state-transition.md) for the verification flow and [Payments](../design/payments.md) for the two payment paths.

The `payload` is opaque to the pool — it is passed to `helper.solve()` unchanged. Its encoding is defined by whichever Helper is used; the reference Helper takes the trade's sides and magnitude.

## sync

```solidity
function sync() external;
```

The permissionless poke: accrues [funding](../protocol/funding-rate.md) at the TWAP, persists the state, and flushes the pending outbox — the protocol cut to `FEE_TO`, the rest to the pool's `PROVIDER`.

## Quoting

`View` is the read-only mirror of the pool's evaluation, including pending funding, for exact off-chain estimates. For close quotes, the pool itself exposes:

```solidity
function getAmountOut(uint256 sideIn, uint256 sideOut, uint256 amountIn, bool minOut)
    external returns (uint256 amountOut);
```

quoting a position (`sideIn` = LONG `0x10` or SHORT `0x20`) against the reserve leg, at the lower or upper oracle bound (`minOut`). Position token ids are specified in [Derivative Tokens](../design/derivative-tokens.md).

## Config

A pool is fully described by its immutable config — which also determines its address (see [Technical Design](../design/README.md)):

```solidity
struct Config {
    address FETCHER;      // oracle logic; address(0) = the built-in Uniswap v3 fetcher
    bytes32 ORACLE;       // packed: quote-token index, TWAP window, pair address
    address TOKEN_R;      // the reserve = settlement token
    uint256 K;            // twice the leverage power (prices are square roots)
    uint256 MARK;         // √(mark price)
    uint256 INTEREST_HL;  // interest half-life in seconds; 0 = off
    uint256 PREMIUM_HL;   // premium half-life in seconds; 0 = off
    uint256 OPEN_RATE;    // opening-fee rate (x128); Q128 = no fee
    address PROVIDER;     // the funding payee (the Vault); 0 = yield to FEE_TO
}
```

Because on-chain prices are carried in square-root form (the Uniswap v3 convention), `MARK` is the square root of the mark price and `K` is twice the pay-off exponent: a `K = 4` pool is a power-2 (×2 compound leverage) market.

`FEE_TO` and `FEE_RATE` — the protocol-fee destination and rate — are immutables on the shared pool logic contract, not per-pool config.
