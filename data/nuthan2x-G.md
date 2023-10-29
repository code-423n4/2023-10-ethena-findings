
| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | additional warm sload and memory expansion | 2 |
| [GAS-2](#GAS-2) | update the state after validating the returned bool | 1 |

### <a name="GAS-1"></a>[GAS-1] [`EthenaMinting::_setMaxRedeemPerBlock()`](https://github.com/code-423n4/2023-10-ethena/blob/d361ced0f57005a1b80c7d01673e17582847aaed/protocols/USDe/contracts/EthenaMinting.sol#L443) Gas inefficient due to additional warm sload and memory expansion.

## Impact
- In `EthenaMinting._setMaxMintPerBlock()`, there is a new local variable `oldMaxMintPerBlock` is declared to cache the storage to emit later after updating it. 
- But it can be avoided and by emitting the params and storage before modifyng the storage. It is implemented at gas efficiently at [SingleAdminAccessControl._grantRole](https://github.com/code-423n4/2023-10-ethena/blob/d361ced0f57005a1b80c7d01673e17582847aaed/protocols/USDe/contracts/SingleAdminAccessControl.sol#L74), but missing at `EthenaMinting._setMaxRedeemPerBlock()` & `EthenaMinting._setMaxRedeemPerBlock()`

## Recommended Mitigation Steps
- In [`EthenaMinting._setMaxRedeemPerBlock()`](https://github.com/code-423n4/2023-10-ethena/blob/d361ced0f57005a1b80c7d01673e17582847aaed/protocols/USDe/contracts/EthenaMinting.sol#L443) function
```solidity diff

  function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {
-    uint256 oldMaxRedeemPerBlock = maxRedeemPerBlock;
    maxRedeemPerBlock = _maxRedeemPerBlock;
-    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);
  }

  function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {
+   emit MaxRedeemPerBlockChanged(maxRedeemPerBlock, _maxRedeemPerBlock);
    maxRedeemPerBlock = _maxRedeemPerBlock;
  }

```
### <a name="GAS-2"></a>[GAS-2] `EthenaMinting._deduplicateOrder()`, update the state only if `valid`== true. 
- `valid` is validated after updating storage, so if valid == false, then its a gas inefficient.

## Recommended Mitigation Steps
- In [`EthenaMinting._setMaxRedeemPerBlock()`](https://github.com/code-423n4/2023-10-ethena/blob/d361ced0f57005a1b80c7d01673e17582847aaed/protocols/USDe/contracts/EthenaMinting.sol#L443) function
```diff

  function _deduplicateOrder(address sender, uint256 nonce) private returns (bool) {
    (bool valid, uint256 invalidatorSlot, uint256 invalidator, uint256 invalidatorBit) = verifyNonce(sender, nonce);
-    mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
-    invalidatorStorage[invalidatorSlot] = invalidator | invalidatorBit;
    return valid;
  }

  function _deduplicateOrder(address sender, uint256 nonce) private returns (bool) {
    (bool valid, uint256 invalidatorSlot, uint256 invalidator, uint256 invalidatorBit) = verifyNonce(sender, nonce);
+    if(valid) {
+        mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
+        invalidatorStorage[invalidatorSlot] = invalidator | invalidatorBit;
+    }
    return valid;
  }
