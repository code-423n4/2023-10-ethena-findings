## [L-01] Users must still wait for cooldown period even after it is set to zero
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L78

In `StakedUSDeV2`, users will not be able to call `unstake(address)` to unstake their USDe if the cooldown period is decreased to a length that would have resulted in their assets being cooled down. Instead, they must wait the original cooldown period.

If the `cooldownDuration` is accidentally set to a large amount, there is no way for this to be corrected. Any users who started a cooldown during this period of a high `cooldownDuration` will need to wait that duration before unstaking.

This is low severity for two reasons:
- The user may be able to shorten their cooldown period if they have additional assets to cooldown and call `cooldownAssets` or `cooldownShares`, depending on if the new value of `cooldownDuration` + `block.timestamp` is shorter than the mistaken value of `cooldownDuration`
- A mistakenly large value of `cooldownDuration` is limited to `MAX_COOLDOWN_DURATION` of 90 days

The recommendation is to store `cooldownStart` instead of `cooldownEnd` in the `UserCooldown` struct. The cooldown end can than be calculated in `unstake` based on the current value of `cooldownDuration`.

```solidity
block.timestamp >= userCooldown.cooldownStart + cooldownDuration
```

## [L-02] Unnecessary call to `getUnvestedAmount`

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L89-L91
```solidity
function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
    uint256 newVestingAmount = amount + getUnvestedAmount();

	...
}
```

In `StakedUSDe.transferInRewards`, there is an unnecessary call to `getUnvestedAmount()` that will always result in 0 due to the conditional on the previous line. The type is an unsigned integer and the function will revert if it is ever greater than zero. Therefore, the value here will always be equal to zero.