# API

A state transition on a Derion pool is done by calling the `swap` function on the pool contract. The same function can also be statically called to perform input and output estimation.

```solidity
function swap(
    Param memory param,
    Payment memory payment
) returns (uint256 amountIn, uint256 amountOut, uint256 price)

struct Param {
    uint256 sideIn;
    uint256 sideOut;
    address helper;
    bytes payload;
}

struct Payment {
    address utr;
    bytes payer;
    address recipient;
}
```

## Sides

There's 3 sides in a pool:

* SIDE\_LONG = 0x10
* SIDE\_SHORT = 0x20
* SIDE\_LP = 0x30

`sideIn` and `sideOut` represent the state transistion types:

* opening Long/Short/LP position will have `sideIn` = 0
* closing Long/Short/LP position will have `sideOut` = 0

## Helper

* helper: Helper contract address
* payload: data will be passed to `Helper.swapToState` contract call internally by the `swap` function.
* amountIn: is the maximum amount of sideIn token that will be spent in the swap transaction

```solidity
// encode payload
bytes memory payload = abi.encode(sideIn, sideOut, amountIn);
// decode payload
(sideIn, sideOut, amountIn) = abi.decode(payload, (uint256, uint256, uint256));
```

Note: `sideIn` and `sideOut` must be passed twice: first in the `Param`, and another time packed in Helper's `payload`.

## Payment

`recipient` is the target account that the token output will be transfered to.

There are 2 methods of payments:

### Direct Token Approval

If `payer` is an empty bytes, the `utr` param will be ignored, and `swap` require a direct approval from `msg.sender` to transfer the `sideIn` token in. This mode is recommended for inter-contract interaction only.

### ERC-6120

If `payer` is the 20 bytes address of the paying account, the `swap` call is expected to be invoked over an `ERC-6120`'s execution by that `payer` account.

The `utr` is the address of the ERC-6120 contract that invoke this call.

```javascript
utr.exec([
    // amountOutMin can be configured here
], [{
    inputs: [{
        mode: PAYMENT,
        eip: 20,
        token: tokenIn,
        id: 0,
        amountIn: amountInMax,
        recipient: pool.address,
    }],
    code: pool.address,
    data: (await pool.populateTransaction.swap({
        sideIn: 0,
        sideOut: SIDE_LONG, // open Long
        helper: helper.address,
        payload: abiCoder.encode(
            ["uint", "uint", "uint"],
            [0, SIDE_LONG, amountIn],
        ),
    }, {
        utr: utr.address,
        payer: signer.address,
        recipient: recipient,
    })).data,
}])
```
