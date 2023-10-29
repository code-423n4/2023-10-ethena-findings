## Description

[`transferInRewards` function in `StakedUSDe` wrongly calls and adds `getUnvestedAmount` function in Line 91](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L91)

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
if `getUnvestedAmount()` returns any value other than zero, first line of function will invoke `revert StillVesting();`. Thus, calling function `getUnvestedAmount();` and adding its return value (that will be zero if revert was not invoked in first line) to amount consumes unnecessary Gas.

## Solution
Remove `getUnvestedAmount();` from second line.