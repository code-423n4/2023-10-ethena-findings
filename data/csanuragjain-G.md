https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L91

`getUnvestedAmount()` will always be `0` in `transferInRewards` function. This means below checks are not required and could be revised

```diff
function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
--    uint256 newVestingAmount = amount + getUnvestedAmount();

--    vestingAmount = newVestingAmount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

--    emit RewardsReceived(amount, newVestingAmount);
++    emit RewardsReceived(amount, amount);
  }
```