1. calling the function `getUnvestedAmount()` one time will save gas

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L89-L99

```solidity
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting(); ------------->> 1st time
    uint256 newVestingAmount = amount + getUnvestedAmount(); -------->> 2nd time


    vestingAmount = newVestingAmount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);


    emit RewardsReceived(amount, newVestingAmount);
  }
```