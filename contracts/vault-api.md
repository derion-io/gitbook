# Vault API

One Vault per reserve token. Every function that touches a pool has a second overload taking `bytes oracleData`, forwarded to every pool touch in the call for pools on pull oracles ([Price Oracle](../protocol/oracle/README.md)); one multi-feed blob serves them all.

## Capital

```solidity
function deposit(uint256 amount, Payment memory payment) external payable returns (uint256 shares);
function withdraw(uint256 shares, address helper) external returns (uint256 amount);

function totalNAV(bool minMark) external returns (uint256 nav);
function idleReserve() external view returns (uint256);
function positionOf(address pool) external view returns (uint256);
```

`deposit` settles funding on every active pool, reads NAV at the higher close bound, pulls `amount` (native, Permit2, or `transferFrom`; see [Payments](../design/payments.md)), and mints `VaultLP` to `payment.recipient` pro-rata. `withdraw` settles funding, reads NAV at the lower close bound, burns the caller's shares, closes LP positions to raise any shortfall, and pays exactly the marked amount or reverts. `helper` is the untrusted solver for those closes; every close is floored at the pool's adverse-bound quote. `totalNAV(true)` is the withdraw mark, `totalNAV(false)` the deposit mark. See [Deposit & Withdraw](../liquidity/deposit-withdraw.md).

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

`rebalance` moves `pool` toward its target `2·max(rA, rB)·(1 + headRoom)`: an LP-class open with idle when below, capped by the ramp; a partial close of the Vault's own position when above, floored at the inflection. `index` is the pool's position in the active strategy and is verified. `rate` is a Q64 fraction of the drift in `(0, 1.5]`; `Q64` lands exactly on the target, larger values overshoot by up to half the drift, larger asks are clamped. `defund` closes the Vault's whole position in a pool that `strategy` (the active one, or a candidate with a strict majority) does not fund; `gapIndex` is the pool's sorted insertion point in that blob. `poke`/`pokeAll` call `sync()` on one or all active pools. See [Depth Provision](../liquidity/depth-provision.md).

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
```

`pools` must be strictly ascending, at most 32; `rampHLs` in seconds, each positive (clamped up to `MIN_RAMP_HL` on use); `headRooms` in Q16 (`65536` = 100%). Votes are escrowed `VaultLP`, weighted by amount × time since lock; a strategy is promoted when its weight is a strict majority and the Vault holds nothing in the pools it drops. `stakedFor` and `totalLocked` are raw amounts; the time-weighted comparison is internal. See [Strategy Governance](../liquidity/governance.md).

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
