# QA Report

## Summary

### Non-critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [[N&#x2011;01](#n01-incorrect-error-message)] | Incorrect error message is used when adding a supported asset or custodian address that has already been added | 2 |
| [[N&#x2011;02](#n02-incorrect-function-description)] | The function description of `addToBlacklist()` and `removeFromBlacklist()` are incorrect, or the intended code has not been implemented. | 2 |
| [[N&#x2011;03](#n03-no-need-to-check-address(usde))] | The likelihood of adding `address(usde)` as a custodian address is very low. | 1 |
| [[N&#x2011;04](#n04-wrong-description)] | Wrong description of `vestingAmount` in `StakedUSDe` contract. | 1 |
| [[N&#x2011;05](#n05-code-duplication-can-be-reduced)] | Code duplication can be reduced by creating a separate function for repeated code blocks. | 2 |
| [[N&#x2011;06](#n06-unnecessary-code)] | `newVestingAmount` always equal to `amount` in `StakedUSDe#transferInRewards()` | 1 |


## Non-critical Issues

### [N&#x2011;01] Incorrect error message is used when adding a supported asset or custodian address that has already been added.

`addSupportedAsset()` and `addCustodianAddress()` in `EthenaMinting.sol` are designed to whitelist supported assets and custodian addresses. However, there is an issue with the error messages when attempting to add an asset or address that already exists. Both functions currently display invalid address messages in such cases, which is incorrect.

*There are 2 instances of this issue:*

```solidity

File: contracts/EthenaMinting.sol

290:  function addSupportedAsset(address asset) public onlyRole(DEFAULT_ADMIN_ROLE) {
291:@>  if (asset == address(0) || asset == address(usde) || !_supportedAssets.add(asset)) {
292:      revert InvalidAssetAddress();

298:  function addCustodianAddress(address custodian) public onlyRole(DEFAULT_ADMIN_ROLE) {
299:@>  if (custodian == address(0) || custodian == address(usde) || !_custodianAddresses.add(custodian)) {
300:      revert InvalidCustodianAddress();

```
*GitHub*: [290](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L290C1-L292C36), [298](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L298C1-L300C40)

#### Recommendation

Include a distinct error message for cases where the address has already been added.

```diff
   function addSupportedAsset(address asset) public onlyRole(DEFAULT_ADMIN_ROLE) {
-    if (asset == address(0) || asset == address(usde) || !_supportedAssets.add(asset)) {
+    if (asset == address(0) || asset == address(usde)) {
       revert InvalidAssetAddress();
     }
+    if(!_supportedAssets.add(asset)) {
+      revert AddressAlreadyAdded();
+    }
     emit AssetAdded(asset);
   }
 
   /// @notice Adds an custodian to the supported custodians list.
   function addCustodianAddress(address custodian) public onlyRole(DEFAULT_ADMIN_ROLE) {
-    if (custodian == address(0) || custodian == address(usde) || !_custodianAddresses.add(custodian)) {
+    if (custodian == address(0) || custodian == address(usde)) {
       revert InvalidCustodianAddress();
     }
+    if(!_custodianAddresses.add(custodian)) {
+      revert AddressAlreadyAdded();
+    }
     emit CustodianAddressAdded(custodian);
   }
```

### [N&#x2011;02] The function description of `addToBlacklist()` and `removeFromBlacklist()` are incorrect, or the intended code has not been implemented.

The description of `addToBlacklist()` and `removeFromBlacklist()` says that these operation can be done by owner (DEFAULT_ADMIN_ROLE) or blacklist managers, but the implemented code only allow blacklist managers to do these operations. So the intended code is not implemented or these functions using wrong description.

*There are 2 instances of this issue:*

```solidity

File: contracts/StakedUSDe.sol

102:@> * @notice Allows the owner (DEFAULT_ADMIN_ROLE) and blacklist managers to blacklist addresses.
103:   * @param target The address to blacklist.
104:   * @param isFullBlacklisting Soft or full blacklisting level.
105:   */
106:  function addToBlacklist(address target, bool isFullBlacklisting)
107:    external
108:    onlyRole(BLACKLIST_MANAGER_ROLE)
109:    notOwner(target)
110:  {

116:@> * @notice Allows the owner (DEFAULT_ADMIN_ROLE) and blacklist managers to un-blacklist addresses.
117:   * @param target The address to un-blacklist.
118:   * @param isFullBlacklisting Soft or full blacklisting level.
119:   */
120:  function removeFromBlacklist(address target, bool isFullBlacklisting)
121:    external
122:    onlyRole(BLACKLIST_MANAGER_ROLE)
123:    notOwner(target)
124:  {

```
*GitHub*: [102](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L102C3-L110C4),[116](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L116C3-L124C4)

### [N&#x2011;03] The likelihood of adding `address(usde)` as a custodian address is very low.

The probability of adding `address(usde)` as a custodian address is quite low, and it is on par with any other ERC addresses. Therefore, there is no need to specifically check for `address(usde)` in the `EthenaMinting#addCustodianAddress()` function.

*There are 1 instances of this issue:*

```solidity
File: contracts/EthenaMinting.sol

298:  function addCustodianAddress(address custodian) public onlyRole(DEFAULT_ADMIN_ROLE) {
299: @> if (custodian == address(0) || custodian == address(usde) || !_custodianAddresses.add(custodian)) {
300:      revert InvalidCustodianAddress();

```
*GitHub*: [298](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L298C2-L301C6)

### [N&#x2011;04] Wrong description of `vestingAmount` in `StakedUSDe` contract.

`vestingAmount` gets updated upon the completion of the previous vesting duration. This ensures that, at that moment, there is no remaining unvested amount.

*There are 1 instances of this issue:*

```solidity
File: contracts/StakedUSDe.sol

40:  /// @notice The amount of the last asset distribution from the controller contract into this
41:@>/// contract + any unvested remainder at that time @audit > there will not be any unvested remainder at that time 
42:  uint256 public vestingAmount;

```
*GitHub*: [40](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L40C3-L42C32)


### [N&#x2011;05] Code duplication can be reduced by creating a separate function for repeated code blocks.

Code duplication can often be reduced by creating a separate function for repeated code blocks. This can also make the code more readable and easier to maintain.

*There are 2 instances of this issue:*

```solidity

File: contracts/StakedUSDeV2.sol

100:    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
101:    cooldowns[owner].underlyingAmount += assets;
102:
103:    _withdraw(_msgSender(), address(silo), owner, assets, shares);

116:    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
117:    cooldowns[owner].underlyingAmount += assets;
118:
119:    _withdraw(_msgSender(), address(silo), owner, assets, shares);
```
*GitHub*: [100](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L100C5-L103C67), [116](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L116C1-L119C67)

#### Recommendation

Create a separate function for the cooldown.

```diff
 function cooldownAssets(uint256 assets, address owner) external ensureCooldownOn returns (uint256) {
    if (assets > maxWithdraw(owner)) revert ExcessiveWithdrawAmount();
     uint256 shares = previewWithdraw(assets);
 
-    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
-    cooldowns[owner].underlyingAmount += assets;
-
-    _withdraw(_msgSender(), address(silo), owner, assets, shares);
+    _cooldown(assets, shares, owner);
 
     return shares;
   }

 function cooldownShares(uint256 shares, address owner) external ensureCooldownOn returns (uint256) {
    if (shares > maxRedeem(owner)) revert ExcessiveRedeemAmount();
 
     uint256 assets = previewRedeem(shares);
-    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
-    cooldowns[owner].underlyingAmount += assets;
-
-    _withdraw(_msgSender(), address(silo), owner, assets, shares);
+    _cooldown(assets, shares, owner);

     return assets;
   }

+  function _cooldown(uint256 assets, uint256 shares, address owner) internal {
+    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
+    cooldowns[owner].underlyingAmount += assets;
+
+    _withdraw(_msgSender(), address(silo), owner, assets, shares);
+  }

```

### [N&#x2011;06] `newVestingAmount` always equal to `amount` in `StakedUSDe#transferInRewards()`

The `transferInRewards` function is used to transfer rewards from the controller contract into the `StakedUSDe` contract. The `getUnvestedAmount()` function will always return zero in the code due to the preceding check (`if (getUnvestedAmount() > 0) revert StillVesting();`). Therefore, adding `getUnvestedAmount()` to the `amount` in the line `uint256 newVestingAmount = amount + getUnvestedAmount();` is unnecessary. 

There are 1 instances:

```solidity

File : contracts/StakedUSDe.sol

  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
@>  uint256 newVestingAmount = amount + getUnvestedAmount();

@>  vestingAmount = newVestingAmount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, newVestingAmount);
  }
```

*GitHub*: [89](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L89C1-L99C4)

#### Recommendation 

```diff
   function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
     if (getUnvestedAmount() > 0) revert StillVesting();
-    uint256 newVestingAmount = amount + getUnvestedAmount();
 
-    vestingAmount = newVestingAmount;
+    vestingAmount = amount;
     lastDistributionTimestamp = block.timestamp;
     // transfer assets from rewarder to this contract
     IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);
 
-    emit RewardsReceived(amount, newVestingAmount);
+    emit RewardsReceived(amount, vestingAmount);
   }

```