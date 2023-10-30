# Delete `getUnvestedAmount()` to save gas

## Description
- https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L91

There is a redundant function call to `getUnvestedAmount()` in the `transferInRewards()` function of the StakedUSDe.sol contract. The `getUnvestedAmount()` is equal to zero at `uint256 newVestingAmount = amount + getUnvestedAmount();` because `transferInRewards()` will always revert if `getUnvestedAmount() > 0`.

```solidity
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
    uint256 newVestingAmount = amount + getUnvestedAmount();

    vestingAmount = newVestingAmount;
    ...
  }
```
## Impact
Wasting gas.

## Recommended mitigation steps:
- Using `uint256 newVestingAmount = amount;` instead of `uint256 newVestingAmount = amount + getUnvestedAmount();`
