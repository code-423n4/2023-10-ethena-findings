1. Incorrect Calculation in the `transferInRewards()`

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L89C1-L99C4

The `transferInRewards()` function calculates the `newVestingAmount` by adding the `amount` parameter and the current reward cycle's unvested amount. However, the function's first line already validates that the unvested amount must be equal to 0. Therefore, adding the value of `getUnvestedAmount()` here is unnecessary. Moreover, to achieve the function's intended purpose, the formula in this context should be: `uint256 newVestingAmount = amount + vestingAmount`, ensuring that at least the `vestingAmount` number of USDe tokens are transferred into the contract after the completion of each reward cycle.
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




2. It is recommended to remove the `notOwner(target)` restriction in `removeFromBlacklist()` function

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L120C1-L127C4

In the `removeFromBlacklist()` function, the requirement for the input parameter target is that it should not be the `owner`. However, in extreme cases, a potential `owner` can be blacklisted before becoming the actual `owner`. Therefore, it is advisable to remove the restriction of `notOwner(target)` in case of such scenarios.

```
  function removeFromBlacklist(address target, bool isFullBlacklisting)
    external
    onlyRole(BLACKLIST_MANAGER_ROLE)
    notOwner(target)
  {
    bytes32 role = isFullBlacklisting ? FULL_RESTRICTED_STAKER_ROLE : SOFT_RESTRICTED_STAKER_ROLE;
    _revokeRole(role, target);
  }
```