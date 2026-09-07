# Custom Fetchers

Any contract can be a pool's oracle. It implements two functions:

```solidity
interface IFetcher {
    function fetch(uint256 ORACLE, bytes memory data) external payable returns (uint256 twap, uint256 spot);
    function spot(uint256 ORACLE) external returns (uint256 spotPrice);
}
```

### fetch

`ORACLE` is the pool's config word, which the fetcher interprets however it likes. The convention is to keep the source or asset address in the low 160 bits: the pool emits that address as the `index` topic of every Position event, so indexers can group markets by it.

`data` is the caller's `oracleData`, forwarded verbatim. It is empty on initialization, and on pokes and quotes unless the caller passes some. A read-only source ignores it. A pull oracle applies or parses it before reading. The fetcher must make sure that caller-supplied data can only make the price fresher: verify signatures, bound the publish time, and never let a caller choose among stale values.

Both returned prices are Q128 fixed point in the same convention, and that convention decides how `K` and `MARK` are read: square-root prices give $$k = K/2$$ and a square-root `MARK`; plain prices give $$k = K$$ and a plain `MARK`. A source with a single price returns it as both TWAP and spot.

`fetch` is payable so a paid update can be funded. The pool forwards native value only when `data` is non-empty. A fetcher that charges must spend at most what the update costs and refund the remainder to `msg.sender` in the same call; the pool recovers the refund for the reserve leg or returns it to the caller. The pool forwards value whenever `data` is non-empty, whether or not the fetcher charges, so a fee-less fetcher must refund the whole forward; anything it keeps is charged to the caller as the oracle fee. Pokes, quotes, and view paths are not payable and forward nothing, so a fee-charging source must accept empty data there.

### spot

The seeding price for `init`. It should be the local spot alone, with no external-oracle requirement: only the creator's own dead-share seed is split at this price, so a mispriced spot cannot harm a third party. This is what lets a pool be created before a pull oracle has ever been fed.

### Failing closed

A fetcher revert halts the pool's trades, quotes, and pokes until the condition clears. That is the intended shape: fail closed, never return a wrong price. A pool whose fetcher breaks permanently is stuck with positions that cannot be closed, which is one reason the config, and therefore the fetcher, is part of the pool's address and cannot be swapped.

### Containment

A fetcher can lie to its own pool, and a lying price can misprice that pool's trades against its own holders. It cannot reach the reserve of any other pool, mint any other pool's tokens, or touch the shared token beyond its own pool's ids. The damage a hostile fetcher can do is bounded to the participants of the pool that chose it.
