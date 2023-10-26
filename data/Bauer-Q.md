## Redundant addition operation

The function `StakedUSDe.transferInRewards()` is used to deposit rewards into the contract, update the vesting amount.The issue here is that the calculation of `newVestingAmount` includes the addition of the specified amount to the result of `getUnvestedAmount()`. However, the function reverts if getUnvestedAmount() is greater than 0, meaning the code will never proceed when getUnvestedAmount() is greater than 0. This renders the addition of amount unnecessary and could potentially lead to an undesired state change in the contract when there are unvested amounts.
```solidity
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
    uint256 newVestingAmount = amount + getUnvestedAmount();

    vestingAmount = newVestingAmount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, newVestingAmount);
  }

```