QA0:

verifyRoute should `revert` if ordertype differ  , because if  orderType is not correct the verify logic will bypass  

```solidity 
function verifyRoute(Route calldata route, OrderType orderType) public view override returns (bool) {
    // routes only used to mint
    if (orderType == OrderType.REDEEM) {
-       return true;
+       revert();   
    }
...


```


QA1: 
in function `transferInRewards` , getUnvestedAmount() is always zero, no need to add again.

```solidty

  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();

  -  uint256 newVestingAmount = amount + getUnvestedAmount();
  + uint256 newVestingAmount = amount;
    vestingAmount = newVestingAmount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, newVestingAmount);
  }

```

 



QA2：

`StakedUSDeV2` # `setCooldownDuration ` should Check new duration differ from old  one



```solidity
 function setCooldownDuration(uint24 duration) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (duration > MAX_COOLDOWN_DURATION) {
      revert InvalidCooldown();
    }
    uint24 previousDuration = cooldownDuration;
    cooldownDuration = duration;
    emit CooldownDurationUpdated(previousDuration, cooldownDuration);
  }
```


QA3：

`EthenaMinting` # `hashOrder` better have chain id.

```solidity 
 function hashOrder(Order calldata order) public view override returns (bytes32) {
    return ECDSA.toTypedDataHash(getDomainSeparator(), keccak256(encodeOrder(order)));
  }
```
