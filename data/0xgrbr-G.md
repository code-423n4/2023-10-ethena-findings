# [G-01] Change variable to immutable

1 instance: 

- Change variable [StakedUSDeV2::MAX_COOLDOWN_DURATION](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L22) to immutable

```
@@ -19,7 +19,7 @@ contract StakedUSDeV2 is IStakedUSDeCooldown, StakedUSDe {
 
   USDeSilo public silo;
 
-  uint24 public MAX_COOLDOWN_DURATION = 90 days;
+  uint24 immutable public MAX_COOLDOWN_DURATION = 90 days;
```

Gain:
```
test_constructor() (gas: -3642 (-0.095%)) 
test_fails_v2_if_set_duration_zero() (gas: -117 (-0.504%)) 
testSetCooldown_fuzz(uint24) (gas: -117 (-0.514%)) 
testSetCooldown_zero() (gas: -117 (-0.529%)) 
testSetCooldown_error_gt_max() (gas: -2117 (-13.768%)) 
```

# [G-02] Remove useless call and calculation
On function [StakedUSDe::transferInRewards](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L89) remove a useless calculation. The line `if (getUnvestedAmount() > 0) revert StillVesting()` is already testing if the value is greater, reverting if it is, so the next line: `uint256 newVestingAmount = amount + getUnvestedAmount();` is useless. 
This calculation can be safely removed.

```
@@ -88,14 +88,13 @@ contract StakedUSDe is SingleAdminAccessControl, ReentrancyGuard, ERC20Permit, E
    */
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

Gain:
```
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: -447 (-0.063%)) 
testUSDeValuePerStUSDe() (gas: -455 (-0.086%)) 
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: -457 (-0.087%)) 
testFuzzFairStakeAndUnstakePrices(uint256,uint256,uint256,uint256,uint256) (gas: -463 (-0.088%)) 
testFairStakeAndUnstakePrices() (gas: -455 (-0.108%)) 
test_constructor() (gas: -4208 (-0.110%)) 
testUSDeValuePerStUSDe() (gas: -455 (-0.110%)) 
testUSDeValuePerStUSDe() (gas: -454 (-0.111%)) 
testFairStakeAndUnstakePrices() (gas: -454 (-0.127%)) 
testFairStakeAndUnstakePrices() (gas: -455 (-0.128%)) 
testStakingAndUnstakingBeforeAfterReward() (gas: -455 (-0.134%)) 
testStakingAndUnstakingBeforeAfterReward() (gas: -454 (-0.165%)) 
testStakingAndUnstakingBeforeAfterReward() (gas: -454 (-0.166%)) 
testOwnerCanChangeRewarder() (gas: -568 (-0.220%)) 
testOwnerCanChangeRewarder() (gas: -568 (-0.220%)) 
testOwnerCanChangeRewarder() (gas: -568 (-0.220%)) 
testCannotTransferRewardsWhileVesting() (gas: -455 (-0.306%)) 
testCannotTransferRewardsWhileVesting() (gas: -455 (-0.306%)) 
testFuzzNoJumpInVestedBalance(uint256) (gas: -455 (-0.318%)) 
testFuzzNoJumpInVestedBalance(uint256) (gas: -455 (-0.318%)) 
testFuzzNoJumpInVestedBalance(uint256) (gas: -455 (-0.318%)) 
testCanTransferRewardsAfterVesting() (gas: -642 (-0.326%)) 
testCanTransferRewardsAfterVesting() (gas: -642 (-0.326%)) 
testTransferRewardsFailsInsufficientBalance() (gas: -666 (-0.451%)) 
testTransferRewardsFailsInsufficientBalance() (gas: -666 (-0.451%)) 
testTransferRewardsFailsInsufficientBalance() (gas: -666 (-0.451%)) 
```
# [G-03] Change declaration order will optimize gas on reverts.
Declaring the variable inside the IF will optimize gas on reverts at [StakedUSDeV2.sol::unstake](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L78)

```
@@ -77,9 +77,9 @@ contract StakedUSDeV2 is IStakedUSDeCooldown, StakedUSDe {
   /// @param receiver Address to send the assets by the staker
   function unstake(address receiver) external {
     UserCooldown storage userCooldown = cooldowns[msg.sender];
-    uint256 assets = userCooldown.underlyingAmount;
 
     if (block.timestamp >= userCooldown.cooldownEnd) {
+      uint256 assets = userCooldown.underlyingAmount;
       userCooldown.cooldownEnd = 0;
       userCooldown.underlyingAmount = 0;
```

# [G-04] Short-circuits

There is an opportunity to short-circuit some IFs:

3 instances:

[EthenaMinting::verifyRoute](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L357)
Change the order of the IFs
```
@@ -354,10 +354,10 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
       return true;
     }
     uint256 totalRatio = 0;
-    if (route.addresses.length != route.ratios.length) {
+    if (route.addresses.length == 0) {
       return false;
     }
-    if (route.addresses.length == 0) {
+    if (route.addresses.length != route.ratios.length) {
       return false;
     }
     for (uint256 i = 0; i < route.addresses.length; ++i) {
```

[EthenaMinting::_transferCollateral](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L413)
Change the order of the IFs
```
@@ -418,10 +418,10 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
     uint256[] calldata ratios
   ) internal {
     // cannot mint using unsupported asset or native ETH even if it is supported for redemptions
-    if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();
+    if (asset == NATIVE_TOKEN || !_supportedAssets.contains(asset)) revert UnsupportedAsset();
     IERC20 token = IERC20(asset);
     uint256 totalTransferred = 0;
```

[EthenaMinting::_transferCollateral](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L413)
Cache addresses length (not just for the loop, but for the safeTransferFrom function too)
```
@@ -421,6 +421,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
     if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();
     IERC20 token = IERC20(asset);
     uint256 totalTransferred = 0;
+    uint addressesLength = addresses.length;
     for (uint256 i = 0; i < addresses.length; ++i) {
       uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
       token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
@@ -428,7 +429,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
     }
     uint256 remainingBalance = amount - totalTransferred;
     if (remainingBalance > 0) {
-      token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);
+      token.safeTransferFrom(benefactor, addresses[addressesLength - 1], remainingBalance);
     }
   }
```