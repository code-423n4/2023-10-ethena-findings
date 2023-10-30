### Low 01 - Unnecessary math operation and local variable declaration
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
Since `if` statement would ensure `getUnvestedAmount()` return zero, directly assign `vestingAmount = amount`.

### Low 02 - Emitting duplicated values is unnecessary.
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
(1) Change to `emit RewardsReceived(amount)`;
(2) Or if the intention is to allow adding previously unvested amount to amount, then remove `if` statement.

### Low 3 - Balance redistribution may cause loss of funds
In StakedUSDe.sol - `redistributeLockedAmount()`, when the input `to` argument is `address(0)`, the function will not correctly handle it. The function will not revert but will continue to burn tokens causing permanent loss of funds. And base on the function doc, the intended behavior should be funds transfer, not permanent burning of funds on any occasion.

```solidity
// contracts/StakedUSDe.sol
  /**
   * @dev Burns the full restricted user amount and mints to the desired owner address.
   * @param from The address to burn the entire balance, with the FULL_RESTRICTED_STAKER_ROLE
   * @param to The address to mint the entire balance of "from" parameter.
   */
  function redistributeLockedAmount(address from, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && !hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {
      uint256 amountToDistribute = balanceOf(from);
      _burn(from, amountToDistribute);
|>    if (to != address(0)) _mint(to, amountToDistribute);
      emit LockedAmountRedistributed(from, to, amountToDistribute);
    } else {
      revert OperationNotAllowed();
    }
  }
```
(https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L153)

As seen above, when the admin redistributes funds from a fully restricted staker to another address, when `to` is incorrectly set as zero, the function will not revert, instead it will simply burn the entire balance. 

Recommendations:
Revert when `to` is `address(0)`.


