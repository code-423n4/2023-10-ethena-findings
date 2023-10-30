# GAS Report

## Summary

|ID|Issue|Instances|
|:--:|:---|:--:|
| [[G&#x2011;01]](#g01-redundant-code) | The `notOwner` modifier in the `removeFromBlacklist()` function is redundant. | 1 |
| [[G&#x2011;02]](#g02-unnecesary-code) | Adding `getUnvestedAmount()` to the `amount` in `StakedUSDe#transferInRewards()` is not required | 1 |
| [[G&#x2011;03]](#g03-use-private-constant) | Using private constant for `MAX_COOLDOWN_DURATION` saves deployment gas in `StakedUSDeV2.sol` | 1 |

## Gas Optimizations

### [G&#x2011;03] The `notOwner` modifier in the `removeFromBlacklist()` function is redundant. 

Since the `addToBlacklist` function already prevents the `owner` from being added to the blacklist, there's no scenario where the `owner` would need to be removed from the blacklist.

There are 1 instances:

- *StakedUSDe.sol* ( [#L120](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L120C1-L123C21)):

```solidity

120:  function removeFromBlacklist(address target, bool isFullBlacklisting)
121:    external
122:    onlyRole(BLACKLIST_MANAGER_ROLE)
123: @> notOwner(target)

```

```diff
   function removeFromBlacklist(address target, bool isFullBlacklisting)
     external
     onlyRole(BLACKLIST_MANAGER_ROLE)
-    notOwner(target)
   {
     bytes32 role = isFullBlacklisting ? FULL_RESTRICTED_STAKER_ROLE : SOFT_RESTRICTED_STAKER_ROLE;
     _revokeRole(role, target);
```

#### Gas Savings for ```StakedUSDe.removeFromBlacklist```, obtained via protocol’s tests: Avg `129` gas and deployment cost `26432` gas

```
Before change

| contracts/StakedUSDe.sol:StakedUSDe contract |                 |       |        |       |         |
|----------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                              | Deployment Size |       |        |       |         |
| 3215153                                      | 17592           |       |        |       |         |
| Function Name                                | min             | avg   | median | max   | # calls |
| removeFromBlacklist                          | 2843            | 2847  | 2847   | 2851  | 2       |

After change

| contracts/StakedUSDe.sol:StakedUSDe contract |                 |       |        |       |         |
|----------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                              | Deployment Size |       |        |       |         |
| 3188721                                      | 17460           |       |        |       |         |
| Function Name                                | min             | avg   | median | max   | # calls |
| removeFromBlacklist                          | 2714            | 2718  | 2718   | 2722  | 2       |

```

### [G&#x2011;04] Adding `getUnvestedAmount()` to the `amount` in `StakedUSDe#transferInRewards()` is not required

The `transferInRewards` function is used to transfer rewards from the controller contract into the `StakedUSDe` contract. The `getUnvestedAmount()` function will always return zero in the code due to the preceding check (`if (getUnvestedAmount() > 0) revert StillVesting();`). Therefore, adding `getUnvestedAmount()` to the `amount` in the line `uint256 newVestingAmount = amount + getUnvestedAmount();` is unnecessary. 

There are 1 instances:

- *StakedUSDe.sol* ( [#L89](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L89C1-L99C4)):

```solidity

  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
@>  uint256 newVestingAmount = amount + getUnvestedAmount();

@>  vestingAmount = newVestingAmount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, newVestingAmount);
  }
```

```diff
   function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
     if (getUnvestedAmount() > 0) revert StillVesting();
-    uint256 newVestingAmount = amount + getUnvestedAmount();
 
-    vestingAmount = newVestingAmount;
+    vestingAmount = amount;
     lastDistributionTimestamp = block.timestamp;
     // transfer assets from rewarder to this contract
     IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);
 
-    emit RewardsReceived(amount, newVestingAmount);
+    emit RewardsReceived(amount, vestingAmount);
   }

```


#### Gas Savings for ```StakedUSDe.transferInRewards```, obtained via protocol’s tests: Avg `321` gas and deployment cost `4208` gas

```

Before change

| contracts/StakedUSDe.sol:StakedUSDe contract |                 |       |        |       |         |
|----------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                              | Deployment Size |       |        |       |         |
| 3188721                                      | 17460           |       |        |       |         |
| Function Name                                | min             | avg   | median | max   | # calls |
| transferInRewards                            | 4359            | 42190 | 44596  | 66916 | 14      |

After change

| contracts/StakedUSDe.sol:StakedUSDe contract |                 |       |        |       |         |
|----------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                              | Deployment Size |       |        |       |         |
| 3184513                                      | 17439           |       |        |       |         |
| Function Name                                | min             | avg   | median | max   | # calls |
| transferInRewards                            | 4359            | 41869 | 44141  | 66461 | 14      |

```

### [G&#x2011;05] Using private constant for `MAX_COOLDOWN_DURATION` saves deployment gas in `StakedUSDeV2.sol`


There are 1 instances:

- *StakedUSDe.sol* ( [#L22](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L22)):

```solidity

22:  uint24 public MAX_COOLDOWN_DURATION = 90 days;

```

```diff
-  uint24 public MAX_COOLDOWN_DURATION = 90 days;
+  uint24 private constant MAX_COOLDOWN_DURATION = 90 days;

```


#### Gas Savings for ```StakedUSDe.transferInRewards```, obtained via protocol’s tests: Deployment cost `20525` gas

```

Before change

| contracts/StakedUSDeV2.sol:StakedUSDeV2 contract |                 |       |        |       |         |
|--------------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                                  | Deployment Size |       |        |       |         |
| 3743352                                          | 20352           |       |        |       |         |


After change

| contracts/StakedUSDeV2.sol:StakedUSDeV2 contract |                 |       |        |       |         |
|--------------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                                  | Deployment Size |       |        |       |         |
| 3722827                                          | 20197           |       |        |       |         |

```

