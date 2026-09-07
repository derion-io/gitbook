# Pool API

## transition

All trades, deposits into and withdrawals from the LP class, flips, and rotations go through one function on the pool:

```solidity
function transition(Param memory param, Payment memory payment)
    external payable
    returns (int256 dA, int256 dB, int256 dC, int256 dR, uint256 price);

struct Param {
    int256 dAMin;       // floor on δ(your LONG balance)
    int256 dBMin;       // floor on δ(your SHORT balance)
    int256 dCMin;       // floor on δ(your LP balance)
    int256 dRMin;       // floor on the reserve you receive
    address helper;     // untrusted solver contract
    bytes payload;      // intent, decoded by the helper
    bool unwrapETH;     // deliver received reserve as native ETH
    bytes oracleData;   // optional oracle update, forwarded verbatim to the fetcher
}

struct Payment {
    address recipient;  // where minted tokens / paid-out reserve go (required)
    address owner;      // Permit2 signer; on the signature path MUST equal recipient
    uint256 nonce;      // Permit2 nonce
    uint256 deadline;   // Permit2 deadline
    bytes signature;    // empty → transferFrom(msg.sender); non-empty → Permit2
}
```

Every delta is signed from the caller's perspective: positive = received, negative = given. The returns are the realized deltas and the TWAP the trade was marked at.

The four minimums are pure slippage floors: a receiver floors what it gets (`> 0`), a giver caps what it gives (`< 0`), and `type(int256).min` means no floor. They carry no direction information; every gate runs at both oracle bases regardless ([Price Oracle](../protocol/oracle/README.md)). See [State Transition](../protocol/state-transition.md) for the verification flow and [Payments](../design/payments.md) for the two payment paths.

`payload` is opaque to the pool and passed to `helper.solve()` unchanged. The reference Helper takes:

| Operation | `payload` |
| --- | --- |
| open, close, flip, LP deposit, LP withdraw | `abi.encode(uint256 sideIn, uint256 sideOut, uint256 amount)` |
| rotate (open one side, close the other) | `abi.encode(type(uint256).max, uint256 openSide, uint256 openReserve, uint256 closeSide, uint256 closeAmount, uint256 0)` |

with sides `SIDE_R = 0x00`, `SIDE_A = 0x10`, `SIDE_B = 0x20`, `SIDE_C = 0x30`. `SIDE_E = 0x01` is `SIDE_R` paid out as native ETH; the quoting functions accept it as `sideOut` on a close, as an alias of `SIDE_R`, while a transition makes the same choice with `unwrapETH`. For an open (`SIDE_R → A/B`) `amount` is the **net** reserve; the opening fee is added on top and pulled as part of `dR`. For an LP deposit (`SIDE_R → C`) it is the reserve paid. For closes, withdrawals, and flips it is the number of shares to burn.

`oracleData` is empty for a read-only fetcher. On a pull-oracle pool it carries the signed update blobs, and `msg.value` may carry the update fee; the pool forwards native value to the fetcher only when `oracleData` is non-empty, and what the fetcher does not spend is wrapped toward the reserve owed or returned ([Payments](../design/payments.md)).

### Closing by transfer

```solidity
IERC1155(token).safeTransferFrom(owner, pool, id, amount, abi.encode(param));
```

The pool's `onERC1155Received` decodes the `Param`, runs `transition` with the recipient pinned to the sender, pays out, and refunds any part of the transferred position the close did not consume. Batch transfers are rejected.

The reference Helper accepts the same transfer as a relay: `safeTransferFrom(owner, helper, id, amount, abi.encode(pool, recipient, abi.encode(param)))` makes it forward the position to `pool`, unwrap the WETH it is paid, and send native ETH to `recipient`. It predates `unwrapETH`, which does the same on a direct transfer. Because the pool sees the Helper as the sender, any refund of an unconsumed remainder lands on the Helper, and `sweep(id, recipient)` moves such a balance out for whoever calls it first.

## Helper interface

```solidity
interface IHelper {
    function solve(uint256 openRate, Slippable calldata before, bytes calldata payload)
        external view returns (Proposal memory);
}

struct Slippable {
    uint256 xk;    // (TWAP / MARK)^K, the funding basis
    uint256 xkS;   // the same at the spot; equals xk when the oracle did not diverge
    uint256 R;     // engine reserve
    uint256 rA;    // LONG reserve at the TWAP
    uint256 rB;    // SHORT reserve at the TWAP
    uint256 rAS;   // LONG reserve at the spot
    uint256 rBS;   // SHORT reserve at the spot
    uint256 sA;    // LONG supply
    uint256 sB;    // SHORT supply
    uint256 sC;    // LP supply
}

struct Proposal {
    uint256 a;     // LONG coefficient after the trade
    uint256 b;     // SHORT coefficient after the trade
    int256 dA;     // δ(transactor's LONG)
    int256 dB;     // δ(transactor's SHORT)
    int256 dC;     // δ(transactor's LP)
    int256 dR;     // reserve the transactor receives; the pool sets R1 = R − dR
}
```

The pool calls the transactor's `helper` once, with the post-funding snapshot at both bases and the pool's `OPEN_RATE`, and takes the complete proposal back. The LP reserve at each basis is derivable as $$R - r_A - r_B$$. Everything returned is re-priced and verified by the pool ([State Transition](../protocol/state-transition.md)); a dishonest proposal reverts. Anyone can deploy a Helper and name it per call.

## init

```solidity
function init(State memory state, Payment memory payment)
    external payable
    returns (uint256 rA, uint256 rB, uint256 rC);

struct State {
    uint256 R;   // reserve deposited = the initial engine reserve
    uint256 a;   // LONG coefficient
    uint256 b;   // SHORT coefficient
}
```

Seeds a freshly deployed pool. `R`, `a`, and `b` must be positive and below $$2^{224}$$. The reserve is pulled through the `Payment` path, the two curves are evaluated at the fetcher's spot price, and all three classes must clear the minimum reserve (currently 10⁶ wei each). An insolvent seed, whose two sides already exceed `R` at that price, fails the residual computation outright. The three seeds are minted to `address(1)` and can never be withdrawn. See [Pool Creation](../guide/pool-creation.md).

## sync

```solidity
function sync() external returns (uint256 xk, uint256 rA, uint256 rB, uint256 price);
function sync(bytes memory oracleData) external returns (uint256 xk, uint256 rA, uint256 rB, uint256 price);
```

The permissionless poke: accrues [funding](../protocol/funding-rate.md) at the TWAP, persists the recovered coefficients, and flushes the outbox (the protocol's cut plus any donations) to `FEE_TO`. Returns the post-funding snapshot. Nothing of the LP's depends on it being called.

## Quoting

```solidity
function getAmountOut(uint256 sideIn, uint256 sideOut, uint256 amountIn, bool minOut)
    external returns (uint256 amountOut);
function getAmountOut(uint256 sideIn, uint256 sideOut, uint256 amountIn, bool minOut, bytes memory oracleData)
    external returns (uint256 amountOut);
```

Quotes one leg against the pool at one bound of the TWAP/spot pair. An open (`sideIn = SIDE_R`, `sideOut` a class) returns the shares minted for `amountIn` of net reserve, before the opening fee; a close (`sideIn` a class, `sideOut = SIDE_R` or `SIDE_E`) returns the reserve for `amountIn` shares. `minOut = true` picks the bound adverse to the caller: the class dear on an open (fewest shares), cheap on a close (lowest payout). For the LP class the bounds are compared on the realized residual, since it is not monotone in price. The quote does not accrue pending funding; `View` does.

## State

```solidity
function getStates() external view returns (uint256 R, uint256 a, uint256 b, uint32 lastTime);
function loadConfig() external pure returns (Config memory);
function ensureStateIntegrity() external view;   // reverts while a transition is in progress
function FEE_TO() external view returns (address);      // protocol-fee destination, a logic immutable
function FEE_RATE() external view returns (uint256);    // 1/FEE_RATE of the funding release; 0 = no cut
```

`R` is the engine reserve; the pool's token balance in excess of it is the pending protocol cut. A pool also answers the built-in fetcher's `fetch` and `spot`, which it inherits, and the ERC-1155 receiver hooks used by the transfer-close path.

## Config

A pool is fully described by its immutable config, which also determines its address ([Technical Design](../design/README.md)):

```solidity
struct Config {
    address FETCHER;      // oracle logic; address(0) = the built-in Uniswap v3 / Chainlink fetcher
    bytes32 ORACLE;       // packed oracle word, interpreted by the fetcher
    address TOKEN_R;      // the reserve = settlement token
    uint256 K;            // the exponent applied to the fetcher's price
    uint256 MARK;         // the mark price, in the fetcher's price convention (Q128)
    uint256 INTEREST_HL;  // interest half-life in seconds; 0 = off
    uint256 PREMIUM_HL;   // premium half-life in seconds; 0 = off
    uint256 OPEN_RATE;    // opening-fee rate (x128); 2^128 or 0 = no fee
}
```

`K` and `MARK` follow the fetcher's price convention:

| Fetcher | Prices | `K` | `MARK` |
| --- | --- | --- | --- |
| built-in, Uniswap v3 branch; `CompositeFetcher` | square-root | `2k` | √(mark price) |
| built-in, Chainlink branch; `RobinhoodFetcher` | plain | `k` | mark price |

where $$k$$ is the pay-off power traders see. There is no provider field: the pool knows no LP by name. `FEE_TO` and `FEE_RATE`, the protocol-fee destination and rate, are immutables on the shared pool logic contract, not per-pool config.

## Events

```solidity
event Position(
    address indexed payer,      // payment.owner when set, otherwise msg.sender
    address indexed recipient,  // mint: the recipient; burn: address(0)
    address indexed index,      // the low 160 bits of ORACLE
    uint256 id,                 // (side << 160) | pool
    uint256 amount,             // shares minted or burned
    uint256 price,              // the TWAP the transition was marked at
    uint256 valueR              // the leg's mark-to-curve reserve value at that TWAP
);

event Sync(uint256 R, uint256 price, uint256 xk, uint256 rA, uint256 rB);
```

One `Position` event is emitted per settled leg, so a position is fully described by its own events; there is no separate transition event. `valueR` is `|δ| · r / s` on the post-trade class, the indexer's cost basis, not the cash that moved (it differs from `dR` by the spread and fee). LP-class legs emit it like the sides, so decoders must handle the `0x30` id. The three seeds minted at initialization are the exception: they appear only as ERC-1155 transfers to `address(1)`, not as `Position` events.

`Sync` is the post-funding state snapshot, emitted by every poke and before every trade. It fully describes the reserve state: `a = v(xk, rA, R)`, `b = v(1/xk, rB, R)`, `rC = R − rA − rB`, and the pending protocol cut is `balanceOf(pool) − R`.

## Factory

```solidity
function createPool(Config memory config) external returns (address pool);

function deploy(
    Config memory config, State memory state, Payment memory payment,
    address baseToken, bytes32 baseSymbol, bytes32 topic2, bytes32 topic3
) external payable returns (address pool);

function deployWithStrategy(
    Config memory config, State memory state, Payment memory payment,
    address baseToken, bytes32 baseSymbol, bytes32 topic2, bytes32 topic3,
    address vault, address baseStrategy, uint256 rampHL, uint256 headRoom
) external payable returns (address pool, address strategy);

function create(Config memory config) external returns (address pool);   // createPool without the revert; zero on failure
address public immutable LOGIC;                                         // the shared pool logic
```

A pool's address is the CREATE2 address, with salt 0 and the Factory as deployer, of the MetaProxy init code built from `LOGIC` and `abi.encode(config)`; `MetaProxyView.computeBytecodeHash` produces that hash. `createPool` deploys the MetaProxy only and emits nothing. `deploy` also runs `init` in the same transaction: on a WETH pool `msg.value` is either zero, with the seed pulled through `payment`, or exactly `state.R`; on any other reserve it must be zero. Since `init` pulls from its caller, a `deploy` that sends no native value needs a Permit2 signature in `payment`; the direct-allowance path would pull from the Factory itself. `deployWithStrategy` additionally asks `vault` to write a strategy extending `baseStrategy` (or a fresh single-pool one when it is zero) with the new pool, which LPs can then vote for ([Strategy Governance](../liquidity/governance.md)). `deploy` and `deployWithStrategy` emit an anonymous log with four topics (`baseToken`, `baseSymbol`, `topic2`, `topic3`) whose data is the ABI-encoded config fields followed by the pool address; this is how indexers discover pools, and a pool created with `createPool` is invisible to them.

## Token

```solidity
function mint(address to, uint256 id, uint256 amount, bytes memory data) external;   // only address(uint160(id))
function burn(address from, uint256 id, uint256 amount) external;                     // only address(uint160(id))
function totalSupply(uint256 id) external view returns (uint256);
function uri(uint256 id) external view returns (string memory);      // delegated to the descriptor
function setDescriptor(address descriptor) external;                 // descriptor setter only
function setDescriptorSetter(address setter) external;               // descriptor setter only
```

The shared ERC-1155 is an OpenZeppelin `ERC1155Supply` named `Derion Position` (`DERION-POS`), plus the open mint/burn rule ([Derivative Tokens](../design/derivative-tokens.md)). `uri` forwards to the descriptor's `constructMetadata(id)`, which the reference descriptor answers only for ids whose low 160 bits resolve to a pool of its Factory (`NOT_A_DERION_TOKEN` otherwise).

## Reverts

Pool, Factory, and Helper checks are `require` strings. Not every revert carries one: a proposal that cannot be applied at all (a burn larger than a class's supply, an insolvent seed at `init`) fails with a Solidity arithmetic panic, a burn or transfer beyond the holder's balance surfaces as the token's OpenZeppelin custom error (`ERC1155InsufficientBalance`), and a malformed `Param` or payload fails in ABI decoding.

| Reason | When |
| --- | --- |
| `PoolBase: ZERO_RECIPIENT`, `SIG_RECIPIENT` | no recipient; the recipient differs from the Permit2 signer |
| `PoolBase: ALREADY_INITIALIZED`, `ZERO_PARAM`, `STATE_OVERFLOW_R/A/B`, `MINIMUM_RESERVE_A/B/C` | `init` on a seeded pool; a zero seed value; a seed value at or above 2^224; a class seeded below 10⁶ wei (an insolvent seed, whose sides already exceed `R`, panics on the residual before this check) |
| `PoolBase: ONLY_TOKEN`, `WRONG_POOL`, `BATCH_NOT_SUPPORTED` | a transfer-close not sent by the token, carrying another pool's id, or sent as a batch |
| `PoolBase: STATE_INTEGRITY` | `ensureStateIntegrity()` called during a transition |
| `PoolLogic: SLIPPAGE` | a realized delta below its floor |
| `PoolLogic: CHARGE`, `INVARIANT_A/B/C` | a value gate failed at either basis |
| `PoolLogic: STATE1_OVERFLOW_R/A/B` | a proposed post-trade value at or above 2^224 |
| `PoolLogic: SIDE_IN`, `SIDE_OUT` | an unsupported side pair in a quote |
| `Utils: NOT_WETH`, `msg.value > amount` | native value on a non-WETH reserve at `init`; more native than the reserve owed |
| `PoolFactory: CREATE2_FAILED`, `VALUE_MISMATCH`, `UNUSED_VALUE`, `POOL_INIT_FAILED` | a duplicate config; a WETH seed value that is neither zero nor `state.R`; native value on a non-WETH pool; the seed did not reach the pool |
| `Helper: ONLY_TOKEN`, `SAME_SIDE`, `SIDE_IN`, `SIDE_OUT`, `ROTATE_SAME`, `ROTATE_OPEN`, `ROTATE_CLOSE`, `NO_TARGET` | reference Helper: a position with data sent to it by anything but the token; `sideIn == sideOut`; a `sideIn` other than A or B on a close or flip; a `sideOut` other than A or B on an open, other than R on an LP withdrawal, or other than R, A, or B on a close; rotation legs that are equal or not both A or B; an aim that lands a coefficient at zero |
| `UNAUTHORIZED_MINT_BURN`, `UNAUTHORIZED` | token: a mint or burn of an id whose low 160 bits are not the caller; a descriptor change by anyone but the setter |
