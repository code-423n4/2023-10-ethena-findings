## Override `_convertToShares` and `_convertToAssets` if implementing custom protection for donation attacks.
From openzepplin [v4.9](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC4626.sol#L30C2-L37C45), openzepplin uses virtual assets and virtual shares to mitigate the risk of donation attack.
```solidity
function _convertToAssets(uint256 shares, Math.Rounding rounding) internal view virtual returns (uint256) {
        return shares.mulDiv(totalAssets() + 1, totalSupply() + 10 ** _decimalsOffset(), rounding);
    }
```
The drawbacks of this approach is stated by [openzepplin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC4626.sol#L39C4-L43C56):
```solidity
 The drawback of this approach is that the virtual shares do capture (a very small) part of the value being accrued
 * to the vault. Also, if the vault experiences losses, the users try to exit the vault, the virtual shares and assets
 * will cause the first user to exit to experience reduced losses in detriment to the last users that will experience
 * bigger losses. Developers willing to revert back to the pre-v4.9 behavior just need to override the
 * `_convertToShares` and `_convertToAssets` functions.
```
The protocol implements custom donation attack protection mechanism they could avoid this drawback by overriding `_convertToShares` to `_convertToAssets` to pre v4.9 implementation.

## Transfer out fund within assets before removal
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L259
```solidity
function removeSupportedAsset(address asset) external onlyRole(DEFAULT_ADMIN_ROLE) {//@audit transfer all 
@>    if (!_supportedAssets.remove(asset)) revert InvalidAssetAddress();
    emit AssetRemoved(asset);
  }
```
funds could be left within these asset.