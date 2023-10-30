### Gas 01 - Waste of gas because of unnecessary math operation and local variable declaration.

In StakedUSDe.sol - `transferInRewards()`, adding zero math operation is performed and the result is assigned to a new local variable.

```solidity
// contracts/StakedUSDe.sol
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
|>  uint256 newVestingAmount = amount + getUnvestedAmount();
    vestingAmount = newVestingAmount;
...
```
(https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L91)

As seen above, `getUnvesteAmount()` would have to return zero to pass the `if` statement, but when `getUnvestedAmount()` is zero, there is no need to add zero to `amount` and delacring `newVestingAmount` which is the same value as `amount`.

Recommendation:
Since `if` statement would ensure `getUnvestedAmount()` return zero, directly assign `vestingAmount = amount`. This saves 6 gas units.

### Gas 02 - Waste of gas due to emitting the same value twice

In StakedUSDe.sol - `transferInRewards()`, `emit RewardsReceived(amount, newVestingAmount)` emits two variables `amount` and `newVestingAmount`. However, `amount` and `newVestingAmount` will always be the same value.

```solidity
// contracts/StakedUSDe.sol
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
    uint256 newVestingAmount = amount + getUnvestedAmount();
...
|>  emit RewardsReceived(amount, newVestingAmount);
```
(https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L98C1-L98C52)

As seen above, when `if` statement passes, `getUnvestedAmount()` returns zero, which means `newVestingAmount` will equal `amount`. And `RewardsReceived` will always emit the same value twice, which is unnecessary.

Recommendation:
Change to `emit RewardsReceived(amount)`, which saves 256 gas units.


