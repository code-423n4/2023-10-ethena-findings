# Report

## Summary

### Low Issues

Total of **2 issues**

|ID|Issue|
|:-:|:--|
| [L-1](#l-1-users-cant-delegate-to-other-users-for-only-redeemmint) |Users can't delegate to other Users for only redeem/mint|
| [L-2](#l-2-grant-role-to-privileged-actors-in-the-constructor-itself)|Grant role to privileged actors in the constructor itself|


### Non-critical Issues

Total of **3 issues**

|ID|Issue|
|:-:|:--|
|[NC-1](#nc-1-zero-value-is-added-can-be-avoided)|Zero value is added, can be avoided|
|[NC-2](#nc-2-unnecessary-calculation-can-be-avoided)|Unnecessary calculation can be avoided|
|[NC-3](#nc-3-add-minterredeemer-function-is-missing)|Add Minter/Redeemer function is missing|

## Low issues


### <a name="l-1-users-cant-delegate-to-other-users-for-only-redeemmint"></a>[L-1] Users can't delegate to other Users for only redeem/mint

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L235C3-L238C4

The `setDelegatedSigner` delegates a delegatee to have power to mint/redeem. This can cause issues when the delegate wants to only delegate the user to perform one operation ie; mint/redeem. Suppose Alice delegates Bob to mint tokens. She only wants Bob to mint tokens for her. Ideal flow will be Bob mints tokens for Alice. But at the same time if Alice has USDe tokens in her account, then Bob can redeem her tokens as long as he is the delegatee.

This is better addressed by having a ENUM like:

```solidity
ENUM ACCESS{
  INVALID, //-0
  MINT, //-1
  REDEEM //2
}
```
And assign it to the `delegatedSigner` mapping instead of true/false
```diff
function setDelegatedSigner(address _delegateTo) external {
-    delegatedSigner[_delegateTo][msg.sender] = true; //Replace with enum, default will be invalid
+    delegatedSigner[_delegateTo][msg.sender] = ACCESS.MINT;
   emit DelegatedSignerAdded(_delegateTo, msg.sender);
  }
```

### <a name="l-2-grant-role-to-privileged-actors-in-the-constructor-itself"></a>[L-2] Grant role to privileged actors in the constructor itself

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L111C2-L147C1
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L70C2-L81C4

In [EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol) and [StakedUSDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol) the roles, `REDEEMER_ROLE`, `MINTER_ROLE`, `GATEKEEPER_ROLE` and  `BLACKLIST_MANAGER_ROLE` are not assigned when the contract is initilalized but by calling `granRole()` externally(as per the tests). It will be better to have the constructor set the roles when the contracts are initilaized as they will be set right away and in any case, wont remain unassigned and cause issues.

In EthenaMinting.sol:

```diff
 constructor(
    IUSDe _usde,
    address[] memory _assets,
    address[] memory _custodians,
    address _admin,
    uint256 _maxMintPerBlock,
    uint256 _maxRedeemPerBlock,
    address minter,
    address redeemer,
    address gateKeeper
  ) {
    ---------------------------
    -------------------------------
    -----------------------------
+ _grantRole(REDEEMER_ROLE, minter );
+ _grantRole(MINTER_ROLE, redeemer);
+ _grantRole(GATEKEEPER_ROLE, gateKeeper);
}
```

In StakesUSDe.sol:

```diff
 constructor(IERC20 _asset, address _initialRewarder, address _owner, address blacklistManager)
    ERC20("Staked USDe", "stUSDe")
    ERC4626(_asset)
    ERC20Permit("stUSDe")
  {
    if (_owner == address(0) || _initialRewarder == address(0) || address(_asset) == address(0)) {
      revert InvalidZeroAddress();
    }

    _grantRole(REWARDER_ROLE, _initialRewarder);
    _grantRole(DEFAULT_ADMIN_ROLE, _owner);
+   _grantRole(BLACKLIST_MANAGER_ROLE, blacklistManager);
  }
```


## Non-Critical issues

### <a name="nc-1-zero-value-is-added-can-be-avoided"></a>[NC-1] Zero value is added, can be avoided

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L89C1-L99C4

The `transferInRewards` in `StakedUSDe.sol` has a redundant '0' added into it. The function first checks `if (getUnvestedAmount() > 0) revert StillVesting()` then in the nect line it adds the `getUnvestedAmount()` into `newUnvestedAmount()`, the uint256 value returned by `getUnvestedAmount()` will be '0' and there is no need to add it and can be removed. 

```diff
function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
-    uint256 newVestingAmount = amount + getUnvestedAmount();
+    uint256 newVestingAmount = amount;

    vestingAmount = newVestingAmount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, newVestingAmount);
  }
```

### <a name="nc-2-unnecessary-calculation-can-be-avoided"></a>[NC-2] Unnecessary calculation can be avoided.

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L173C1-L181C4

The `getUnvestedAmount()` in `StakedUSD.sol` returns the unvestedAmount by running a calculation based on the `lastDistributionTimestamp` and `vestingAmount`, it will return zero after the calculation if vestingAmount is zero. So the code can be refactored instead of performing additional calculation.

```diff
function getUnvestedAmount() public view returns (uint256) {
    uint256 timeSinceLastDistribution = block.timestamp - lastDistributionTimestamp;

-    if (timeSinceLastDistribution >= VESTING_PERIOD) {
+    if(timtimeSinceLastDistribution >= VESTING_PERIOD || vestingAmount == 0 ){
      return 0;
    }

    return ((VESTING_PERIOD - timeSinceLastDistribution) * vestingAmount) / VESTING_PERIOD;
  }
```

### <a name="nc-3-add-minterredeemer-function-is-missing"></a>[NC-3] Add Minter/Redeemer function is missing

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L277C2-L285C4

We can see that in [EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol) there are `removeMinterRole` and `removeRedeemerRole` but never functions defined to add these roles(yeah, they can be assigned by calling granRole() externally but so does revokeRole() ) , either remove these two or add functions for `adding` those roles for convinience.

```diff
+  function addMinterRole(address minter) external onlyRole(GATEKEEPER_ROLE) { //@audit addminter function is missing
+    _grantRole(MINTER_ROLE, minter);
+  }

+  function addRedeemerRole(address redeemer) external onlyRole(GATEKEEPER_ROLE) {
+    _grantRole(REDEEMER_ROLE, redeemer);
+  }
```