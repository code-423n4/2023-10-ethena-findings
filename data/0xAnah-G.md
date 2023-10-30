# ETHENA LABS GAS OPTIMIZATIONS

## INTRODUCTION
The majority of optimizations underwent benchmarking using the protocol's tests, specifically employing the following configuration: `solc version 0.8.19`, optimizer enabled, and `20000` runs. For optimizations that were not subjected to benchmarking, their rationale is elucidated through EVM gas expenses and opcodes.

Highlighted below are optimizations exclusively targeting state-mutating functions and view/pure functions invoked by state-mutating functions. In the discussion that follows, only runtime gas is emphasized, given its inevitable dominance over deployment gas costs throughout the protocol's lifetime. Additionally, certain code excerpts may be abbreviated for brevity, and comments within the code snippets may include @audit tags to facilitate the explanation of issues.



## TABLE OF FINDINGS
| Number |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| G-01| Use constants for state variables whose value is known beforehand and is never changed | 1 | - |
| G-02| Add unchecked blocks for subtractions where the operands cannot underflow | 1 | 77 |
| G-03| Sort Solidity operations using short-circuit mode | 2 | - |
| G-04| The result of a function call should be cached rather than re-calling the function | 1 | 318 |
| G-05| Computations should be memoized rather than having to re-compute them | 1 | - |
| G-06| Move lesser gas costing revert checks to the top | 1 |  |
| G-07| Stack variable used as a cheaper cache for a state variable is only used once | 2 | - |
| G-08| Refactor `StakedUSDeV2.unstake()` to avoid reading from state in some scenarios | 1 | 100 |
| G-09| Use the existing Local variable/global variable when equal to a state variable to avoid reading from state | 3 | 291 |
| G-10| Multiple accesses of a array should use a local variable cache | 2 | - |
| G-11| Unbounded Gas Consumption Risk Due to External Call Recipients | 2 |   |





## [G-01] Use constants for state variables whose value is known beforehand and is never changed
When you declare a state variable as a constant, you're telling the Solidity compiler that this variable's value is known at compile time and will never change during the contract's lifetime. Because of this:
The value of the constant state variable is computed and set at the time of contract deployment, not during runtime.

Since the value is known beforehand and cannot change, there is no need to store this value on the Ethereum blockchain, as it can be computed directly whenever needed.
When you access the constant state variable in your contract functions, it consume very little gas because the value is readily available (saved in the contracts bytecode) and does not require any computation.

So, by using constants for state variables whose value is known beforehand and is static (never changed), you save gas because you avoid unnecessary storage and computation costs associated with regular state variables.

### 1 Instance
1. #### The `MAX_COOLDOWN_DURATION` state variable should be made a constatnt since its value is known before hand and never modified.
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L22
```solidity
file: contracts/StakedUSDeV2.sol

22:  uint24 public MAX_COOLDOWN_DURATION = 90 days;
```
Making the `MAX_COOLDOWN_DURATION` state variable constant (since its value is known before hand and never changed) would help avoid storage related operations and replace them with cheaper stack read `3` gas units thereby reducing deployment costs and gas usage of the `setCooldownDuration()` function that reads the value of `MAX_COOLDOWN_DURATION` . The diff below shows how the code could be refactored:
```diff
diff --git a/contracts/StakedUSDeV2.sol b/contracts/StakedUSDeV2.sol
index df2bb48..f8fa980 100644
--- a/contracts/StakedUSDeV2.sol
+++ b/contracts/StakedUSDeV2.sol
@@ -19,7 +19,7 @@ contract StakedUSDeV2 is IStakedUSDeCooldown, StakedUSDe {

   USDeSilo public silo;

-  uint24 public MAX_COOLDOWN_DURATION = 90 days;
+  uint24 public constant MAX_COOLDOWN_DURATION = 90 days;

   uint24 public cooldownDuration;
```




## [G-02] Add unchecked blocks for subtractions where the operands cannot underflow
There are some checks to avoid an underflow, but in some scenarios where it is impossible for underflow to occur we can use unchecked blocks to have some gas savings.
#### Please note this instances was not included in the bots report.

### 1 Instance
1. #### Its impossible for the subtraction `block.timestamp - lastDistributionTimestamp` to underflow so it can be unchecked
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L174
```solidity
file: contracts/StakedUSDe.sol

173:  function getUnvestedAmount() public view returns (uint256) {
174:    uint256 timeSinceLastDistribution = block.timestamp - lastDistributionTimestamp;  //@audit can be unchecked
175:
176:    if (timeSinceLastDistribution >= VESTING_PERIOD) {
177:      return 0;
178:    }
179:
180:    return ((VESTING_PERIOD - timeSinceLastDistribution) * vestingAmount) / VESTING_PERIOD;
181:  }
```
In the `getUnvestedAmount()` function above it is impossible for the subtraction `block.timestamp - lastDistributionTimestamp` to underflow since the value of the current `block.timestanp` cannot be less than the value of `lastDistributionTimestamp`. The diff below shows how the code could be refactored:
```diff
diff --git a/contracts/StakedUSDe.sol b/contracts/StakedUSDe.sol
index 0a56a7d..c04db6a 100644
--- a/contracts/StakedUSDe.sol
+++ b/contracts/StakedUSDe.sol
@@ -171,7 +171,10 @@ contract StakedUSDe is SingleAdminAccessControl, ReentrancyGuard, ERC20Permit, E
    * @notice Returns the amount of USDe tokens that are unvested in the contract.
    */
   function getUnvestedAmount() public view returns (uint256) {
-    uint256 timeSinceLastDistribution = block.timestamp - lastDistributionTimestamp;
+    uint256 timeSinceLastDistribution;
+    unchecked {
+      timeSinceLastDistribution = block.timestamp - lastDistributionTimestamp;
+    }

     if (timeSinceLastDistribution >= VESTING_PERIOD) {
       return 0;
```
#### Gas saving for `getUnvestedAmount()` function obtained via protocol test: Avg 77 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 837 | 837 |  837 | 8 |
| After | 760 | 760 |  760 | 8 |




## [G-03] Sort Solidity operations using short-circuit mode
Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

//f(x) is a low gas cost operation \
//g(y) is a high gas cost operation 

//Sort operations with different gas costs as follows \
f(x) || g(y) \
f(x) && g(y)

### 2 Instances
1. #### Apply short-circuiting rule to `if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN)`
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L421
```solidity
file: contracts/EthenaMinting.sol

413:  function _transferCollateral(
414:    uint256 amount,
415:    address asset,
416:    address benefactor,
417:    address[] calldata addresses,
418:    uint256[] calldata ratios
419:  ) internal {
420:    // cannot mint using unsupported asset or native ETH even if it is supported for redemptions
421:    if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();
422:    IERC20 token = IERC20(asset);
.
.
.
433:  }
```
In the `_transferCollateral()` function of the `EthenaMinting` contract we can apply the short-circuiting rule to the if  statement `if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN)` by placing the less gas costing computation `asset == NATIVE_TOKEN` before the more gas costing computation `_supportedAssets.contains(asset)` in the if revert statement. Therefore in scenarios where `asset == NATIVE_TOKEN` condition results to `true` the EVM would not need to perform the gas costing operations of `!_supportedAssets.contains(asset)` (which involves reading from state and an external call). The code should be refactored as shown in the diff below:
```diff
diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol
index 32da3a5..8dc955c 100644
--- a/contracts/EthenaMinting.sol
+++ b/contracts/EthenaMinting.sol
@@ -418,7 +418,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
     uint256[] calldata ratios
   ) internal {
     // cannot mint using unsupported asset or native ETH even if it is supported for redemptions
-    if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();
+    if (asset == NATIVE_TOKEN || !_supportedAssets.contains(asset)) revert UnsupportedAsset();
     IERC20 token = IERC20(asset);
     uint256 totalTransferred = 0;
     for (uint256 i = 0; i < addresses.length; ++i) {
```

2. #### Apply short-circuiting rule to `if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && to != address(0))`
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L246
```solidity
file: contracts/StakedUSDe.sol

245:  function _beforeTokenTransfer(address from, address to, uint256) internal virtual override {
246:    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && to != address(0)) {
247:      revert OperationNotAllowed();
248:    }
249:    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {
250:      revert OperationNotAllowed();
251:    }
252:  }
```
In the `_beforeTokenTransfer()` function of the `StakedUSDe` contract we can apply the short-circuiting rule to the if  statement `if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && to != address(0))` by placing the less gas costing computation `to != address(0)` before the more gas costing computation `hasRole(FULL_RESTRICTED_STAKER_ROLE, from)` in the if revert statement. Therefore in scenarios where `to != address(0)` condition results to `false` the EVM would not need to perform the gas costing operations of `hasRole(FULL_RESTRICTED_STAKER_ROLE, from)` (which involves reading from state and an external call). The code should be refactored as shown in the diff below:
```diff
diff --git a/contracts/StakedUSDe.sol b/contracts/StakedUSDe.sol
index 0a56a7d..8551478 100644
--- a/contracts/StakedUSDe.sol
+++ b/contracts/StakedUSDe.sol
@@ -243,7 +243,7 @@ contract StakedUSDe is SingleAdminAccessControl, ReentrancyGuard, ERC20Permit, E
    */

   function _beforeTokenTransfer(address from, address to, uint256) internal virtual override {
-    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && to != address(0)) {
+    if (to != address(0) && hasRole(FULL_RESTRICTED_STAKER_ROLE, from)) {
       revert OperationNotAllowed();
     }
     if (hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {
```




## [G-04] The result of a function call should be cached rather than re-calling the function
The execution of function calls within Solidity could incur significant gas costs. When identical function calls produce the same results and need to be utilized multiple times, it is advisable to employ caching as a means to mitigate gas consumption from repeated calls.

### 1 Instance
1. ### Cache invocation of `getUnvestedAmount()` 
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L90
```solidity
file: contracts/StakedUSDe.sol

89:  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
90:    if (getUnvestedAmount() > 0) revert StillVesting(); //@audit getUnvestedAmount() first invocation
91:    uint256 newVestingAmount = amount + getUnvestedAmount(); //@audit getUnvestedAmount() second invocation
92:
93:    vestingAmount = newVestingAmount;
94:    lastDistributionTimestamp = block.timestamp;
95:    // transfer assets from rewarder to this contract
96:    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);
97:
98:    emit RewardsReceived(amount, newVestingAmount);
99:  }
```
The diff below shows how the code could be refactored:
```diff
diff --git a/contracts/StakedUSDe.sol b/contracts/StakedUSDe.sol
index 0a56a7d..e80db82 100644
--- a/contracts/StakedUSDe.sol
+++ b/contracts/StakedUSDe.sol
@@ -87,8 +87,9 @@ contract StakedUSDe is SingleAdminAccessControl, ReentrancyGuard, ERC20Permit, E
    * @param amount The amount of rewards to transfer.
    */
   function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
-    if (getUnvestedAmount() > 0) revert StillVesting();
-    uint256 newVestingAmount = amount + getUnvestedAmount();
+    uint256 unvestedAmount = getUnvestedAmount();
+    if (unvestedAmount > 0) revert StillVesting();
+    uint256 newVestingAmount = amount + unvestedAmount;

     vestingAmount = newVestingAmount;
     lastDistributionTimestamp = block.timestamp;
```
#### Gas saving for `transferInRewards()` function obtained via protocol test: Avg 318 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 4359 | 66916 |  42190 | 14 |
| After | 4364 | 66455 |  41872 | 14 |




## [G-05] Computations should be memoized rather than having to re-compute them
In computing, memoization or memoisation is an optimization technique used primarily to speed up computer programs by storing the results of expensive computations and returning the cached result when the same inputs occur again.

### 1 Instance 
1. #### We can memoize the computation of `route.addresses.length`.
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L357
```solidity
file: contracts/EthenaMinting.sol

351:  function verifyRoute(Route calldata route, OrderType orderType) public view override returns (bool) {
352:    // routes only used to mint
353:    if (orderType == OrderType.REDEEM) {
354:      return true;
355:    }
356:    uint256 totalRatio = 0;
357:    if (route.addresses.length != route.ratios.length) {    //@audit memoize route.addresses.length
358:      return false;
359:    }
360:    if (route.addresses.length == 0) {    //@audit memoize route.addresses.length
361:      return false;
362:    }
363:    for (uint256 i = 0; i < route.addresses.length; ++i) {    //@audit memoize route.addresses.length
364:      if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)
365:      {
366:        return false;
367:      }
368:      totalRatio += route.ratios[i];
369:    }
370:    if (totalRatio != 10_000) {
371:      return false;
372:    }
373:    return true;
374:  }
```
In the `verifyRoute()` function above the computation `route.addresses.length` was repeated multiple times. We could save the gas used in the subsequent calculations if we memoize the calculation i.e we cache the result of the calcultion the first time in a variable and use the variable in place of the subsequent calculations. The diff below shows how the code could be refactored:
```diff
diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol                                                    
index 32da3a5..2d4a54c 100644                                                                                             
--- a/contracts/EthenaMinting.sol                                                                                         
+++ b/contracts/EthenaMinting.sol                                                                                         
@@ -354,13 +354,14 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu                    
       return true;                                                                                                       
     }                                                                                                                    
     uint256 totalRatio = 0;                                                                                              
-    if (route.addresses.length != route.ratios.length) {                                                                 
+    uint256 routeAddressLen = route.addresses.length;                                                                    
+    if (routeAddressLen != route.ratios.length) {                                                                        
       return false;                                                                                                      
     }                                                                                                                    
-    if (route.addresses.length == 0) {                                                                                   
+    if (routeAddressLen == 0) {                                                                                          
       return false;                                                                                                      
     }                                                                                                                    
-    for (uint256 i = 0; i < route.addresses.length; ++i) {                                                               
+    for (uint256 i = 0; i < routeAddressLen; ++i) {                                                                      
       if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0) 
       {                                                                                                                  
         return false;                                                                                                    
```




## [G-06] Move lesser gas costing revert checks to the top
Revert() statements that check input arguments or cost lesser gas should be at the top of the function.
Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.

### 1 Instance
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L339
```solidity
file: contracts/EthenaMinting.sol

339:  function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
340:    bytes32 taker_order_hash = hashOrder(order);
341:    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
342:    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
343:    if (order.beneficiary == address(0)) revert InvalidAmount();
344:    if (order.collateral_amount == 0) revert InvalidAmount();
345:    if (order.usde_amount == 0) revert InvalidAmount();
346:    if (block.timestamp > order.expiry) revert SignatureExpired();
347:    return (true, taker_order_hash);
348:  }
```
In the `verifyOrder()` function above the revert checks should be re-ordered such that the more gas consuming operation `if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature()` should moved lower so that in scenarios where the less gas consuming operations would fail the `EVM` would not squander gas when the function would eventually fail. The code should be refactored as shown below:
```diff
diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol
index 32da3a5..939a650 100644
--- a/contracts/EthenaMinting.sol
+++ b/contracts/EthenaMinting.sol
@@ -339,11 +339,11 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
   function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
     bytes32 taker_order_hash = hashOrder(order);
     address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
-    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
     if (order.beneficiary == address(0)) revert InvalidAmount();
     if (order.collateral_amount == 0) revert InvalidAmount();
     if (order.usde_amount == 0) revert InvalidAmount();
     if (block.timestamp > order.expiry) revert SignatureExpired();
+    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
     return (true, taker_order_hash);
   }
```





## [G-07] Stack variable used as a cheaper cache for a state variable is only used once
If the variable is only accessed once, it's cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend.
#### Please note the below instances were not included in the bot reports

### 2 Instances
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L381
```solidity
file: contracts/EthenaMinting.sol

377:  function verifyNonce(address sender, uint256 nonce) public view override returns (bool, uint256, uint256, uint256) {
378:    if (nonce == 0) revert InvalidNonce();
379:    uint256 invalidatorSlot = uint64(nonce) >> 8;
380:    uint256 invalidatorBit = 1 << uint8(nonce);
381:    mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
382:    uint256 invalidator = invalidatorStorage[invalidatorSlot];
383:    if (invalidator & invalidatorBit != 0) revert InvalidNonce();
384:
385:    return (true, invalidatorSlot, invalidator, invalidatorBit);
386:  }
```
We can save `3` gas units in the `verifyNonce()` function if we access `_orderBitmaps[sender]` mapping directly rather than assigning it to the local storage variable `invalidatorStorage` which was only used once in the function. The code could be refcatored as shown in the diff below: 
```diff
diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol
index 32da3a5..a06a258 100644
--- a/contracts/EthenaMinting.sol
+++ b/contracts/EthenaMinting.sol
@@ -378,8 +378,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
     if (nonce == 0) revert InvalidNonce();
     uint256 invalidatorSlot = uint64(nonce) >> 8;
     uint256 invalidatorBit = 1 << uint8(nonce);
-    mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
-    uint256 invalidator = invalidatorStorage[invalidatorSlot];
+    uint256 invalidator = _orderBitmaps[sender][invalidatorSlot];
     if (invalidator & invalidatorBit != 0) revert InvalidNonce();

     return (true, invalidatorSlot, invalidator, invalidatorBit);
```


- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L393
```solidity
file: contracts/EthenaMinting.sol

391:  function _deduplicateOrder(address sender, uint256 nonce) private returns (bool) {
392:    (bool valid, uint256 invalidatorSlot, uint256 invalidator, uint256 invalidatorBit) = verifyNonce(sender, nonce);
393:    mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
394:    invalidatorStorage[invalidatorSlot] = invalidator | invalidatorBit;
395:    return valid;
396:  }
```
We can save `3` gas units in the `_deduplicateOrder()` function if we access `_orderBitmaps[sender]` mapping directly rather than assigning it to the local storage variable `invalidatorStorage` which was only used once in the function. The code could be refcatored as shown in the diff below:
```diff
diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol
index 32da3a5..1f3a077 100644
--- a/contracts/EthenaMinting.sol
+++ b/contracts/EthenaMinting.sol
@@ -390,8 +390,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
   /// @notice deduplication of taker order
   function _deduplicateOrder(address sender, uint256 nonce) private returns (bool) {
     (bool valid, uint256 invalidatorSlot, uint256 invalidator, uint256 invalidatorBit) = verifyNonce(sender, nonce);
-    mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
-    invalidatorStorage[invalidatorSlot] = invalidator | invalidatorBit;
+    _orderBitmaps[sender][invalidatorSlot] = invalidator | invalidatorBit;
     return valid;
   }
```





## [G-08] Refactor `StakedUSDeV2.unstake()` to avoid reading from state in some scenarios
In scenarios where `if (block.timestamp >= userCooldown.cooldownEnd)` results to `false` we could avoid executing the statement `uint256 assets = userCooldown.underlyingAmount` since the function would revert. Refactoring the code as shown in the diff below would help correct this.

- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L78-#L90
```solidity
file: 

78:  function unstake(address receiver) external {
79:    UserCooldown storage userCooldown = cooldowns[msg.sender];
80:    uint256 assets = userCooldown.underlyingAmount;
81:
82:    if (block.timestamp >= userCooldown.cooldownEnd) {
83:      userCooldown.cooldownEnd = 0;
84:      userCooldown.underlyingAmount = 0;
85:
86:      silo.withdraw(receiver, assets);
87:    } else {
88:      revert InvalidCooldown();
89:    }
90:  }
```

```diff
diff --git a/contracts/StakedUSDeV2.sol b/contracts/StakedUSDeV2.sol
index df2bb48..924ba36 100644
--- a/contracts/StakedUSDeV2.sol
+++ b/contracts/StakedUSDeV2.sol
@@ -77,13 +77,13 @@ contract StakedUSDeV2 is IStakedUSDeCooldown, StakedUSDe {
   /// @param receiver Address to send the assets by the staker
   function unstake(address receiver) external {
     UserCooldown storage userCooldown = cooldowns[msg.sender];
-    uint256 assets = userCooldown.underlyingAmount;

     if (block.timestamp >= userCooldown.cooldownEnd) {
+
+      silo.withdraw(receiver, userCooldown.underlyingAmount);
       userCooldown.cooldownEnd = 0;
       userCooldown.underlyingAmount = 0;
-
-      silo.withdraw(receiver, assets);
+
     } else {
       revert InvalidCooldown();
     }
```




## [G-09] Use the existing Local variable/global variable when equal to a state variable to avoid reading from state

### 3 Instances
1. #### Use local variable `duration` in place of the state variable `cooldownDuration` in the emit `CooldownDurationUpdated()` since they are equal.
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L133
```solidity
file: contracts/StakedUSDeV2.sol

126:  function setCooldownDuration(uint24 duration) external onlyRole(DEFAULT_ADMIN_ROLE) {
127:    if (duration > MAX_COOLDOWN_DURATION) {
128:      revert InvalidCooldown();
129:    }
130:
131:    uint24 previousDuration = cooldownDuration;
132:    cooldownDuration = duration;
133:    emit CooldownDurationUpdated(previousDuration, cooldownDuration);
134:  }
```
In the `setCooldownDuration()` function above we can avoid 1 `SLOAD(Gwarmaccess)` `100` gas units if we use the `duration` variable in place of the `cooldownDuration` state variable in the emit `CooldownDurationUpdated()` statement since both `duration` and `cooldownDuration` are of equal values: The code should be refactored as shown in the diff below:
```diff
diff --git a/contracts/StakedUSDeV2.sol b/contracts/StakedUSDeV2.sol           
index df2bb48..d3f1b29 100644                                                  
--- a/contracts/StakedUSDeV2.sol                                               
+++ b/contracts/StakedUSDeV2.sol                                               
@@ -130,6 +130,6 @@ contract StakedUSDeV2 is IStakedUSDeCooldown, StakedUSDe { 
                                                                               
     uint24 previousDuration = cooldownDuration;                               
     cooldownDuration = duration;                                              
-    emit CooldownDurationUpdated(previousDuration, cooldownDuration);         
+    emit CooldownDurationUpdated(previousDuration, duration);                 
   }                                                                           
 }                                                                             
```
```
Estimated gas saved: 97 gas units
```


2. #### Use local variable `_maxMintPerBlock` in place of the state variable `maxMintPerBlock` in the emit `MaxMintPerBlockChanged()` since they are equal.
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L439
```solidity
file: contracts/EthenaMinting.sol

436:  function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {
437:    uint256 oldMaxMintPerBlock = maxMintPerBlock;
438:    maxMintPerBlock = _maxMintPerBlock;
439:    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);
440:  }
```
In the `_setMaxMintPerBlock()` function above we can avoid 1 `SLOAD(Gwarmaccess)` `100` gas units if we use the `_maxMintPerBlock` variable in place of the `maxMintPerBlock` state variable in the emit `MaxMintPerBlockChanged()` statement since both `_maxMintPerBlock` and `maxMintPerBlock` are of equal values: The code should be refactored as shown in the diff below:
```diff
diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol
index 32da3a5..f569e40 100644
--- a/contracts/EthenaMinting.sol
+++ b/contracts/EthenaMinting.sol
@@ -436,7 +436,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
   function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {
     uint256 oldMaxMintPerBlock = maxMintPerBlock;
     maxMintPerBlock = _maxMintPerBlock;
-    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);
+    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, _maxMintPerBlock);
   }
```
```
Estimated gas saved: 97 gas units
```


3. #### Use local variable `_maxRedeemPerBlock` in place of the state variable `maxRedeemPerBlock` in the emit `MaxRedeemPerBlockChanged()` since they are equal.
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L446
```solidity
file: contracts/EthenaMinting.sol

443:  function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {
444:    uint256 oldMaxRedeemPerBlock = maxRedeemPerBlock;
445:    maxRedeemPerBlock = _maxRedeemPerBlock;
446:    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);
447:  }
```
In the `_setMaxRedeemPerBlock()` function above we can avoid 1 `SLOAD(Gwarmaccess)` `100` gas units if we use the `_maxRedeemPerBlock` variable in place of the `maxRedeemPerBlock` state variable in the emit `MaxRedeemPerBlockChanged()` statement since both `_maxRedeemPerBlock` and `maxRedeemPerBlock` are of equal values: The code should be refactored as shown in the diff below:
```diff
diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol
index 32da3a5..8198a3e 100644
--- a/contracts/EthenaMinting.sol
+++ b/contracts/EthenaMinting.sol
@@ -443,7 +443,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
   function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {
     uint256 oldMaxRedeemPerBlock = maxRedeemPerBlock;
     maxRedeemPerBlock = _maxRedeemPerBlock;
-    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);
+    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, _maxRedeemPerBlock);
   }
```
```
Estimated gas saved: 97 gas units
```



## [G-10] Multiple accesses of a array should use a local variable cache
The instances below point to the second+ access of a value inside an array, within a function. Caching an array's struct avoids re-calculating the array offsets into memory.
#### Please note the below instances were not included in the bot reports

### Proof of concept 
```solidity
struct Person {
    string name;
    uint age;
    uint id;
}

contract NoCacheArrayElement {

    Person[] students;

    function createStudents() external  {
        Person[] memory arrayOfPersons = new Person[](3); 
        Person memory newPerson1 = Person("Emmanuel", 15,1);
        Person memory newPerson2 = Person("Faustina", 16,2);
        Person memory newPerson3 = Person("Emmanuela", 14,3);

        arrayOfPersons[0] = newPerson1;
        arrayOfPersons[1] = newPerson2;
        arrayOfPersons[2] = newPerson3;

        _addNewSet(arrayOfPersons);
    }

    function _addNewSet(Person[] memory _persons) internal {
        uint len = _persons.length;
        unchecked {
            for(uint i; i < len; ++i) {
                Person memory newStudent = Person(_persons[i].name, _persons[i].age, _persons[i].id);
                students.push(newStudent);
            }
        }

    }
}
```
```
test for test/NoCacheArrayElement.t.sol:NoCacheArrayElementTest
[PASS] test_createStudents() (gas: 230357)
```

```solidity

struct Person {
    string name;
    uint age;
    uint id;
}

contract CacheArrayElement {

    Person[] students;

    function createStudents() external  {
        Person[] memory arrayOfPersons = new Person[](3); 
        Person memory newPerson1 = Person("Emmanuel", 15,1);
        Person memory newPerson2 = Person("Faustina", 16,2);
        Person memory newPerson3 = Person("Emmanuela", 14,3);

        arrayOfPersons[0] = newPerson1;
        arrayOfPersons[1] = newPerson2;
        arrayOfPersons[2] = newPerson3;

        _addNewSet(arrayOfPersons);
    }

    function _addNewSet(Person[] memory _persons) internal {
        uint len = _persons.length;
        unchecked {
            for(uint i; i < len; ++i) {
                Person memory myPerson = _persons[i];
                Person memory newStudent = Person(myPerson.name, myPerson.age, myPerson.id);
                students.push(newStudent);
            }
        }

    }
}
```
```
test for test/Counter.t.sol:CacheArrayElementTest
[PASS] test_createStudents() (gas: 230096)
```

### 2 Instance
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L364
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L368
```solidity
file: contracts/EthenaMinting.sol

351:  function verifyRoute(Route calldata route, OrderType orderType) public view override returns (bool) {
352:    // routes only used to mint
.
.
.
363:    for (uint256 i = 0; i < route.addresses.length; ++i) {
364:      if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)
365:      {
366:        return false;
367:      }
368:      totalRatio += route.ratios[i];
369:    }
370:    if (totalRatio != 10_000) {
371:      return false;
372:    }
373:    return true;
374:  }
```

```diff
diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol                                                    
index 32da3a5..466ae55 100644                                                                                             
--- a/contracts/EthenaMinting.sol                                                                                         
+++ b/contracts/EthenaMinting.sol                                                                                         
@@ -361,11 +361,13 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu                    
       return false;                                                                                                      
     }                                                                                                                    
     for (uint256 i = 0; i < route.addresses.length; ++i) {                                                               
-      if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0) 
+      address routeAddress = route.addresses[i];                                                                         
+      uint256 routeRatio = route.ratios[i];                                                                              
+      if (!_custodianAddresses.contains(routeAddress) || routeAddress == address(0) || routeRatio == 0)                  
       {                                                                                                                  
         return false;                                                                                                    
       }                                                                                                                  
-      totalRatio += route.ratios[i];                                                                                     
+      totalRatio += routeRatio;                                                                                          
     }                                                                                                                    
     if (totalRatio != 10_000) {                                                                                          
       return false;                                                                                                      
```


















## [G-11] Unbounded Gas Consumption Risk Due to External Call Recipients
In the context of Solidity, external function calls without a specified gas limit present a significant risk. The callee contract has the potential to consume all the gas allocated to the transaction, causing an undesired revert and disrupt the function's execution. To mitigate this, it's recommended to explicitly set a gas limit when performing external calls using addr.call{gas: }. This limits the gas forwarded to the callee, preventing potential pitfalls and offering better control over transaction execution.

### 2 Instances
- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L250
```solidity
file: contracts/EthenaMinting.sol

247:  function transferToCustody(address wallet, address asset, uint256 amount) external nonReentrant onlyRole(MINTER_ROLE) {
248:    if (wallet == address(0) || !_custodianAddresses.contains(wallet)) revert InvalidAddress();
249:    if (asset == NATIVE_TOKEN) {
250:      (bool success,) = wallet.call{value: amount}("");
251:      if (!success) revert TransferFailed();
252:    } else {
253:      IERC20(asset).safeTransfer(wallet, amount);
254:    }
255:    emit CustodyTransfer(wallet, asset, amount);
256:  }
```

```diff
diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol
index 32da3a5..3464a59 100644
--- a/contracts/EthenaMinting.sol
+++ b/contracts/EthenaMinting.sol
@@ -244,10 +244,10 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
   }

   /// @notice transfers an asset to a custody wallet
-  function transferToCustody(address wallet, address asset, uint256 amount) external nonReentrant onlyRole(MINTER_ROLE) {
+  function transferToCustody(address wallet, address asset, uint256 amount, uint256 _gas) external nonReentrant onlyRole(MINTER_ROLE) {
     if (wallet == address(0) || !_custodianAddresses.contains(wallet)) revert InvalidAddress();
     if (asset == NATIVE_TOKEN) {
-      (bool success,) = wallet.call{value: amount}("");
+      (bool success,) = wallet.call{value: amount, gas: _gas}("");
       if (!success) revert TransferFailed();
     } else {
       IERC20(asset).safeTransfer(wallet, amount);
```


- https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L404
```solidity
file: contracts/EthenaMinting.sol

401:  function _transferToBeneficiary(address beneficiary, address asset, uint256 amount) internal {
402:    if (asset == NATIVE_TOKEN) {
403:      if (address(this).balance < amount) revert InvalidAmount();
404:      (bool success,) = (beneficiary).call{value: amount}("");
405:      if (!success) revert TransferFailed();
406:    } else {
407:      if (!_supportedAssets.contains(asset)) revert UnsupportedAsset();
408:      IERC20(asset).safeTransfer(beneficiary, amount);
409:    }
410:  }
```

```diff
file: contracts/EthenaMinting.sol

diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol
index 32da3a5..c6128c1 100644
--- a/contracts/EthenaMinting.sol
+++ b/contracts/EthenaMinting.sol
@@ -398,10 +398,10 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
   /* --------------- INTERNAL --------------- */

   /// @notice transfer supported asset to beneficiary address
-  function _transferToBeneficiary(address beneficiary, address asset, uint256 amount) internal {
+  function _transferToBeneficiary(address beneficiary, address asset, uint256 amount, uint256 _gas) internal {
     if (asset == NATIVE_TOKEN) {
       if (address(this).balance < amount) revert InvalidAmount();
-      (bool success,) = (beneficiary).call{value: amount}("");
+      (bool success,) = (beneficiary).call{value: amount, gas: _gas}("");
       if (!success) revert TransferFailed();
     } else {
       if (!_supportedAssets.contains(asset)) revert UnsupportedAsset();
```


















## CONCLUSION
As you embark on the journey of incorporating the recommended changes, we want to emphasize the utmost importance of proceeding with vigilance and dedicating thorough efforts to comprehensive testing. It is of paramount significance to ensure that the proposed alterations do not inadvertently introduce fresh vulnerabilities, while also successfully achieving the anticipated enhancements in performance.

We strongly advise conducting a meticulous and exhaustive evaluation of the modifications made to the codebase. This rigorous scrutiny and exhaustive assessment will play a pivotal role in affirming both the security and efficacy of the refactored code. Your careful attention to detail, coupled with the implementation of a robust testing framework, will provide the necessary assurance that the refined code aligns with your security objectives and effectively fulfills the intended performance optimizations.