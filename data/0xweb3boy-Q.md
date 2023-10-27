# [LOW-1]  There should be different functions for disabling minting and disabling redeeming.
 
 ```solidity
function disableMintRedeem() external onlyRole(GATEKEEPER_ROLE) {
    _setMaxMintPerBlock(0);
    _setMaxRedeemPerBlock(0);
  }
```

The above function once called will disable `Mint` and `Redeem` both at once by setting `_setMaxMintPerBlock(0)` and `_setMaxRedeemPerBlock(0)` but if we want to enable them again we have to do it one by one by calling `setMaxMintPerBlock()` and `setMaxRedeemPerBlock()` one by one.

