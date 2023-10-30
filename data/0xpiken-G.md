## [01] [L91](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L91) in [`transferInRewards()`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L89-L99) is unnecessory because `getUnvestedAmount()` is always zero:
```diff
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
-   uint256 newVestingAmount = amount + getUnvestedAmount();

-   vestingAmount = newVestingAmount;
+   vestingAmount = amount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, amount);
  }
```
## [02] Define [`MAX_COOLDOWN_DURATION`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L22) as constant:
```diff
- uint24 public MAX_COOLDOWN_DURATION = 90 days;
+ uint24 public constant MAX_COOLDOWN_DURATION = 90 days;
```
## [03] The number of calls to `token.safeTransferFrom()` in [`EthenaMinting#_transferCollateral()`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L413-L433) can be reduced by on because the last amount in iteration and the remaining balance (if not zero) will be transferred to the same address:
```diff
  function _transferCollateral(
    uint256 amount,
    address asset,
    address benefactor,
    address[] calldata addresses,
    uint256[] calldata ratios
  ) internal {
    // cannot mint using unsupported asset or native ETH even if it is supported for redemptions
    if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();
    IERC20 token = IERC20(asset);
    uint256 totalTransferred = 0;
-   for (uint256 i = 0; i < addresses.length; ++i) {
+   for (uint256 i = 0; i < addresses.length - 1; ++i) {
      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
      totalTransferred += amountToTransfer;
    }
    uint256 remainingBalance = amount - totalTransferred;
-   if (remainingBalance > 0) {
      token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);
-   }
  }
```