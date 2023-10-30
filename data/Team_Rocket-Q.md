## Low Severity Findings

### [L-01] Users who started the cooldown period have to wait for `cooldownDuration` even if `cooldownDuration` is set to 0. 
When a user calls `cooldownAssets` or `cooldownShares` a cooldown starts and have to wait `cooldownDuration` in order to claim his staking amount. 
When admin sets the `setCooldownDuration` to 0:
- users who entered the cooldown phase are forced to wait the cooldown to finish. 
- the other users can claim their staking amount immediately. 

Suggested fix: check if `cooldownDuration == 0` in `unstake function:

```solidity
  function unstake(address receiver) external {
    UserCooldown storage userCooldown = cooldowns[msg.sender];
    uint256 assets = userCooldown.underlyingAmount;

    if ((block.timestamp >= userCooldown.cooldownEnd) || (cooldownDuration == 0)){
      userCooldown.cooldownEnd = 0;
      userCooldown.underlyingAmount = 0;

      silo.withdraw(receiver, assets);
    } else {
      revert InvalidCooldown();
    }
  }

```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L100

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L116

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L82


## Non-critical Findings

### [NC-01] USDeSilo.sol: unused import

SafeERC20 library is imported by `USDeSilo` but never used. 

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDeSilo.sol#L5C48-L5C48

### [NC-02] StakedUSDe.sol function call has no effect.
Second call to `getUnvestedAmount` from `transferInRewards` can be removed since will always return 0. 

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L91
```solidity
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
   //@audit if next line is executed getUnvestedAmount will return 0 
    uint256 newVestingAmount = amount + getUnvestedAmount();
```