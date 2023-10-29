# [LOW-1]  There should be different functions for disabling minting and disabling redeeming.
 
 ```solidity
function disableMintRedeem() external onlyRole(GATEKEEPER_ROLE) {
    _setMaxMintPerBlock(0);
    _setMaxRedeemPerBlock(0);
  }
```

The above function once called will disable `Mint` and `Redeem` both at once by setting `_setMaxMintPerBlock(0)` and `_setMaxRedeemPerBlock(0)` but if we want to enable them again we have to do it one by one by calling `setMaxMintPerBlock()` and `setMaxRedeemPerBlock()` one by one.

# [LOW-2]  `USDe.sol :: setMinter()` Minter role can be set to zero(0) address. 

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L23-L26

```solidity
function setMinter(address newMinter) external onlyOwner {
    emit MinterUpdated(newMinter, minter);
    minter = newMinter;
  }
```

There is no check for address zero(0) in `setMinter()` function that could lead to setting minter as address 0.
Make sure to place a require condition that doesn't allow setting up address 0 as minter.