# Vault API

One Vault per reserve token. Every function that touches a pool has a second overload taking `bytes oracleData`, forwarded to every pool touch in the call for pools on pull oracles ([Price Oracle](../protocol/oracle/README.md)); one multi-feed blob serves them all. Those overloads work only against a fee-less receiver, because the pool's poke and quote paths forward no value ([Stock Tokens](../protocol/oracle/stock-tokens.md)).

## Capital

```solidity
function deposit(uint256 amount, Payment memory payment) external payable returns (uint256 shares);
function withdraw(uint256 shares, address helper) external returns (uint256 amount);

function totalNAV(bool minMark) external returns (uint256 nav);
function idleReserve() external view returns (uint256);
function positionOf(address pool) external view returns (uint256);

address public immutable TOKEN;      // the shared ERC-1155
address public immutable TOKEN_R;    // the reserve every served pool must use
address public immutable WETH;
```

The Vault contract is itself the `VaultLP` ERC-20. `deposit` settles funding on every active pool, reads NAV at the higher close bound, pulls `amount` (native, Permit2, or `transferFrom`; see [Payments](../design/payments.md)), and mints `VaultLP` to `payment.recipient` pro-rata. `withdraw` settles funding, reads NAV at the lower close bound, burns the caller's shares, closes LP positions to raise any shortfall, and pays exactly the marked amount or reverts. `helper` is the untrusted solver for those closes; every close is floored at the pool's adverse-bound quote. `totalNAV(true)` is the withdraw mark, `totalNAV(false)` the deposit mark. The first deposit must exceed 10⁶ wei; 10⁶ of its shares are minted to a dead address as an inflation guard. See [Deposit & Withdraw](../liquidity/deposit-withdraw.md).

## Depth

```solidity
function rebalance(address pool, uint256 index, uint256 rate, address helper) external;
function defund(address pool, address strategy, uint256 gapIndex, address helper) external;
function poke(address pool) external;
function pokeAll() external;

function lastRebalance(address pool) external view returns (uint256);
function promotedAt() external view returns (uint256);
uint256 public immutable MIN_RAMP_HL;
```

`rebalance` moves `pool` toward its target `2·max(rA, rB)·(1 + headRoom)`: an LP-class open with idle when below, capped by the ramp; a partial close of the Vault's own position when above, floored at the inflection. `index` is the pool's position in the active strategy and is verified. `rate` is a Q64 fraction of the drift in `(0, 1.5]`; `Q64` lands exactly on the target, larger values overshoot by up to half the drift, larger asks are clamped. `defund` closes the Vault's whole position in a pool that `strategy` does not fund, where `strategy` is the active one or a candidate whose raw locked shares are a strict majority of all locked shares, without time weighting; `gapIndex` is the pool's sorted insertion point in that blob, and the pool's ramp clock is cleared afterwards. `poke` calls `sync()` on any pool address; `pokeAll` on every pool in the active set. See [Depth Provision](../liquidity/depth-provision.md).

## Governance

```solidity
function createStrategy(address[] memory pools, uint256[] memory rampHLs, uint256[] memory headRooms) external returns (address strategy);
function extendStrategy(address base, address pool, uint256 rampHL, uint256 headRoom) external returns (address strategy);

function lock(address strategy, uint256 amount) external;
function lockAndVote(address strategy, uint256 amount) external;
function unlock(address strategy, uint256 amount) external;
function revote(address from, address to, uint256 amount) external;
function changeStrategy(address strategy) external;

function activeStrategy() external view returns (address);
function isStrategy(address) external view returns (bool);
function stakedFor(address strategy) external view returns (uint256);
function totalLocked() external view returns (uint256);
function locked(address account, address strategy) external view returns (uint256);
function stakedWS(address strategy) external view returns (uint256);   // Σ amount × lock time behind it
function totalWS() external view returns (uint256);                    // Σ amount × lock time over all locks
function lockedWS(address account, address strategy) external view returns (uint256);
```

`pools` must be strictly ascending, at most 32; `rampHLs` in seconds, each positive and below 2^64 (clamped up to `MIN_RAMP_HL` on use); `headRooms` in Q16 (`65536` = 100%), below 2^32. Votes are escrowed `VaultLP`, weighted by amount × time since lock; a strategy is promoted when its weight is a strict majority of all locked weight and the Vault holds nothing in the pools it drops. `lockAndVote` reverts unless the strategy is active after the lock, which a fresh lock of zero weight can only satisfy for a strategy that already holds the majority. `stakedFor` and `totalLocked` are raw amounts; the weighted comparison at time $$t$$ is $$2\,(\text{stakedFor}\cdot t - \text{stakedWS}) > \text{totalLocked}\cdot t - \text{totalWS}$$, reproducible from the getters above. See [Strategy Governance](../liquidity/governance.md).

## Events

```solidity
event Deposited(address indexed account, uint256 reserveIn, uint256 shares);
event Withdrawn(address indexed account, uint256 shares, uint256 reserveOut);
event Funded(address indexed pool, uint256 reserveIn, uint256 minted);
event Defunded(address indexed pool, uint256 burned, uint256 reserveOut);
event StrategyCreated(address indexed strategy);
event Locked(address indexed account, address indexed strategy, uint256 amount);
event Unlocked(address indexed account, address indexed strategy, uint256 amount);
event Revoted(address indexed account, address indexed from, address indexed to, uint256 amount);
event StrategyChanged(address indexed strategy);
event VaultPool(address indexed pool, bool active);
```

`VaultPool` fires right after `StrategyChanged`, once for every pool the new strategy adds (`true`) or drops (`false`) relative to the outgoing one, in ascending pool order, so the fundable set can be maintained from logs alone. The Vault's own LP positions are ordinary pool `Position` events with the Vault as recipient or payer.

## Reverts

| Reason | When |
| --- | --- |
| `Vault: ZERO_RECIPIENT`, `SIG_RECIPIENT`, `ZERO_AMOUNT`, `MIN_DEPOSIT`, `ZERO_SHARES` | a deposit with no recipient, a recipient other than the signer, a zero amount, a first deposit at or below 10⁶ wei, or too small to mint a share |
| `Vault: ZERO_PAYOUT`, `ILLIQUID` | a withdrawal that would pay nothing; the closes could not raise the marked amount |
| `Vault: INDEX`, `WRONG_INDEX` | `rebalance` with an index outside the active strategy or pointing at another pool |
| `Vault: NO_AUTHORITY`, `IS_FUNDED`, `GAP`, `NOTHING` | `defund` under a strategy without authority, against a pool the strategy funds, with a gap index past the end, or with no position to close |
| `Vault: UNKNOWN_STRATEGY`, `UNKNOWN_BASE` | a vote for, or an extension of, a blob the Vault did not create |
| `Vault: NO_MAJORITY`, `DEFUND_FIRST` | `changeStrategy` or `lockAndVote` without the majority; a promotion while a dropped pool is still held |
| `Strategy: LEN`, `MAX_POOLS`, `ORDER`, `RAMP_HL`, `HEAD_ROOM` | array lengths differ; more than 32 pools; pools not strictly ascending; a ramp half-life of zero or at least 2^64; a headroom at or above 2^32 |
