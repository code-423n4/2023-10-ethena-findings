### L-1 Redundant call of the `getUnvestedAmount` function 
There is `getUnvestedAmount()` call at the line #91 which always returns `0` because of the check at the line #90 in the `transferInRewards` of the `StakedUSDe` contract:
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L90-L91
```solidity
89  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
90    if (getUnvestedAmount() > 0) revert StillVesting();
91    uint256 newVestingAmount = amount + getUnvestedAmount();
```
Consider removing this call.

### L-2 Owner can remove all assets addresses but constructor has check at least one asset exist
There is a check at the constructor if at least one asset exists in the `EthenaMinting` contract. But the `DEFAULT_ADMIN_ROLE` user can remove all assets addresses  
```solidity
120    if (_assets.length == 0) revert NoAssetsProvided();

258  /// @notice Removes an asset from the supported assets list
259  function removeSupportedAsset(address asset) external onlyRole(DEFAULT_ADMIN_ROLE) {
260    if (!_supportedAssets.remove(asset)) revert InvalidAssetAddress();
261    emit AssetRemoved(asset);
262  }
```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L258-L262
Consider adding the check of the minimal `_supportedAssets` length.