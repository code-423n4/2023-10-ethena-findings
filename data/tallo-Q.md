# [L-01] ```StakedUSDe#transferInRewards``` only allows the owner to transfer in additional rewards if the vesting period has passed.
```
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
    uint256 newVestingAmount = amount + getUnvestedAmount();
```

The ```getUnvestedAmount()``` function call will always cause the first conditional to revert if it returns > 0. This means that when the line 
```
    uint256 newVestingAmount = amount + getUnvestedAmount();
```
is reached, ```getUnvestedAmount()``` must equal zero. So ```newVestingAmount``` will always equal ```amount```


Instances: 
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L91

