## [G-01] Function _deduplicateOrder(address sender, uint256 nonce) can be optimized further


https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L391C2-L396C4
File: contracts/EthenaMinting.sol
```
 function _deduplicateOrder(address sender, uint256 nonce) private returns (bool) {
392:      (bool valid, uint256 invalidatorSlot, uint256 invalidator, uint256 invalidatorBit) = verifyNonce(sender, nonce);
393:      mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
          invalidatorStorage[invalidatorSlot] = invalidator | invalidatorBit;
          return valid;
  }
```
We look into the implementation of the function verifyNonce() which is being called in the line 392. The function has the following implementation:
```
function verifyNonce(address sender, uint256 nonce) public view override returns (bool, uint256, uint256, uint256) {
           if (nonce == 0) revert InvalidNonce();
           uint256 invalidatorSlot = uint64(nonce) >> 8;
           uint256 invalidatorBit = 1 << uint8(nonce);
           mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
           uint256 invalidator = invalidatorStorage[invalidatorSlot];
           if (invalidator & invalidatorBit != 0) revert InvalidNonce();
           return (true, invalidatorSlot, invalidator, invalidatorBit);
  }
```
Put the above implementation in the context of our main function and we see a similar line in both functions.
```
393:     mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
```
This means we are making this calls twice, one in the function being called, and the other one by the calling function.

To avoid this, we can inline the function verifyNonce() and refactor the code as follows:
```
function _deduplicateOrder(address sender, uint256 nonce) private returns (bool) {
    if (nonce == 0) revert InvalidNonce();
    uint256 invalidatorSlot = uint64(nonce) >> 8;
    uint256 invalidatorBit = 1 << uint8(nonce);
    mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
    uint256 invalidator = invalidatorStorage[invalidatorSlot];
    if (invalidator & invalidatorBit != 0) revert InvalidNonce();
    invalidatorStorage[invalidatorSlot] = invalidator | invalidatorBit;
    return true;
}
```



## [G-02] Avoid Unnecessary Storage in Function verifyNonce(address sender, uint256 nonce)

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L376C1-L386C1
```solidity
 function verifyNonce(address sender, uint256 nonce) public view override returns (bool, uint256, uint256, uint256) {
           if (nonce == 0) revert InvalidNonce();
           uint256 invalidatorSlot = uint64(nonce) >> 8;
           uint256 invalidatorBit = 1 << uint8(nonce);
381:       mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
382:       uint256 invalidator = invalidatorStorage[invalidatorSlot];
           if (invalidator & invalidatorBit != 0) revert InvalidNonce();
           return (true, invalidatorSlot, invalidator, invalidatorBit);
  }
```

Of interest to us is the statements on line 381 and 382:
```solidity
381: mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
382: uint256 invalidator = invalidatorStorage[invalidatorSlot];
```
Instead Of creating a separate storage variable "invalidatorStorage", directly get the "invalidator" access from the state variable read.
Use the below Code snippet

```solidity
-         mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
-         uint256 invalidator = invalidatorStorage[invalidatorSlot];
+         uint256 invalidator = _orderBitmaps[sender][invalidatorSlot];
```

## [G-03] Function verifyOrder() can be optimized further


https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L339C2-L348C4
```
 function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
        bytes32 taker_order_hash = hashOrder(order);
        address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
        if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
343:    if (order.beneficiary == address(0)) revert InvalidAmount();
344:    if (order.collateral_amount == 0) revert InvalidAmount();
345:    if (order.usde_amount == 0) revert InvalidAmount();
346:    if (block.timestamp > order.expiry) revert SignatureExpired();
        return (true, taker_order_hash);
  }
```
Apply the checks in the line 343-346 in the beginning of the function to save gas