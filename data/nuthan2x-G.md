
[`EthenaMinting::_setMaxRedeemPerBlock()`](https://github.com/code-423n4/2023-10-ethena/blob/d361ced0f57005a1b80c7d01673e17582847aaed/protocols/USDe/contracts/EthenaMinting.sol#L443) Gas inefficient due to additional warm sload and memory expansion.

## Impact
- In `EthenaMinting._setMaxMintPerBlock()`, there is a new local variable `oldMaxMintPerBlock` is declared to cache the storage to emit later after updating it. 
- But it can be avoided and by emitting the params and storage before modifyng the storage. It is implemented at gas efficiently at [SingleAdminAccessControl._grantRole](https://github.com/code-423n4/2023-10-ethena/blob/d361ced0f57005a1b80c7d01673e17582847aaed/protocols/USDe/contracts/SingleAdminAccessControl.sol#L74), but missing at `EthenaMinting._setMaxRedeemPerBlock()` & `EthenaMinting._setMaxRedeemPerBlock()`
- Severity : NA
- Likelihood : NA

## Proof of Concept
NA

## Tools Used
- Manual verification

## Recommended Mitigation Steps
- In [`EthenaMinting._setMaxRedeemPerBlock()`](https://github.com/code-423n4/2023-10-ethena/blob/d361ced0f57005a1b80c7d01673e17582847aaed/protocols/USDe/contracts/EthenaMinting.sol#L443) function
```diff

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