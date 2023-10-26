In StakedUSDe contract there is a function on line https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L91:

```
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

The line `uint256 newVestingAmount = amount + getUnvestedAmount();` got no sence because `newVestingAmount ` will always be equal to `amount`, as the function will revert if `getUnvestedAmount()` greater than 0.

You can save gas by deleting that line: so it will not create a new variable, make a call to the other function and do calculations there. 