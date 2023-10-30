# [G-01] Function `transferInRewards()` can be more optimized.

In the `transferInRewards` there is a check that `getUnvestedAmount` must return 0 otherwise it revert. However, it used again when `newVestingAmount` is calculating. There is no need to use it since it can't return other value than zero.

Hence we can remove usage of `getUnvestedAmount` and function will be more gas optimized.

```solidity
function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
    uint256 newVestingAmount = amount + getUnvestedAmount(); // @audit no need to use `getUnvestedAmount` since it return 0.

    vestingAmount = newVestingAmount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, newVestingAmount);
}
```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L89-L99

## Recommended Mitigation Steps
Consider to remove `getUnvestedAmount()`.

```diff
function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
--  uint256 newVestingAmount = amount + getUnvestedAmount();

--  vestingAmount = newVestingAmount;
++  vestingAmount = amount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

--  emit RewardsReceived(amount, newVestingAmount);
++  emit RewardsReceived(amount);
}
```

Gas benchmarks of `transferInRewards()` function.

|          | avg      | median   | max      |
|----------|----------|----------|----------|
| Before   | 49136    | 49136    | 65316    |
| After    | 48751    | 48751    | 64545    |

## Recommended Mitigation Steps
Consider to implement this optimization.

# [G-02] `MAX_COOLDOWN_DURATION` should be marked as `constant`.

`MAX_COOLDOWN_DURATION` should be marked as `constant` since it doesn't change anywhere.

```diff
--  uint24 public MAX_COOLDOWN_DURATION = 90 days;
++  uint24 private constant MAX_COOLDOWN_DURATION = 90 days;
```

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L22

Contract deployment cost:

|          | Deployment cost |
|----------|-----------------|
| Before   | 3773992         |
| After    | 3753468         |


## Recommendation:
Consider to mark it as `constant` since it cheaper.
