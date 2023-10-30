# [L-01] `EthenaMinting.usde` is unchangable.
File:
    https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L63
    https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDe.sol#L23-L26

According to [USDe.setMinter](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDe.sol#L23-L26), `USDe.minter` can be changed by owner, if the owner of `USDe.sol` is compromised, `USDe.setMinter` can be called to change the `USDe.minter`. If `USDe.minter` is changed, [USDe.mint](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDe.sol#L28C12-L31) will not work. Thus [EthenaMinting.mint](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L178) will not work

# [L-02] StakedUSDe.transferInRewards doesn't need to call getUnvestedAmount while calculating newVestingAmount
File:
    https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L91
According to [StakedUSDe.sol#L90](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L90), `if (getUnvestedAmount() > 0) revert StillVesting()` which means getUnvestedAmount's return value must be __zero__ to continue. In such case, calling `getUnvestedAmount` at [StakedUSDe.sol#L91](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L91) is unnecessary.

# [L-03] While `StakedUSDeV2.cooldownDuration` is changed from non-zero to zero, stakers should be able to call `StakedUSDeV2.unstake` to withdraw assets regardless of `userCooldown.cooldownEnd`
File:
    https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L78-L90
While `StakedUSDeV2.cooldownDuration` is changed from non-zero to zero, a staker can withdraw his asset by calling `StakedUSDeV2.withdraw` or `StakedUSDeV2.redeem`, and those functions transfer stakers' asset immediate.
    But for the stakers who withdraw their asset before `StakedUSDeV2.cooldownDuration` is changed, they have to wait until `userCooldown.cooldownEnd` is reached. I think it's unfair for those stakers.
```diff
diff --git a/contracts/StakedUSDeV2.sol b/contracts/StakedUSDeV2.sol
index df2bb48..84a6c03 100644
--- a/contracts/StakedUSDeV2.sol
+++ b/contracts/StakedUSDeV2.sol
@@ -79,7 +79,7 @@ contract StakedUSDeV2 is IStakedUSDeCooldown, StakedUSDe {
     UserCooldown storage userCooldown = cooldowns[msg.sender];
     uint256 assets = userCooldown.underlyingAmount;
 
-    if (block.timestamp >= userCooldown.cooldownEnd) {
+    if (block.timestamp >= userCooldown.cooldownEnd || cooldownDuration == 0 ) {
       userCooldown.cooldownEnd = 0;
       userCooldown.underlyingAmount = 0;
```
