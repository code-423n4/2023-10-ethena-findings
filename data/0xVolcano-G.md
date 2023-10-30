# Gas report

## Table Of Contents

- [Gas report](#gas-report)
  - [Table Of Contents](#table-of-contents)
  - [Use constants for variables that don't change(Save a storage SLOT: 2200 Gas)](#use-constants-for-variables-that-dont-changesave-a-storage-slot-2200-gas)
  - [Unnecessary SLOADS inside the constructor(Save 2 SLOADS - 4200 Gas)](#unnecessary-sloads-inside-the-constructorsave-2-sloads---4200-gas)
  - [Modifier makes it expensive since we end up reading state twice(Saves 1197 Gas on average from the tests)](#modifier-makes-it-expensive-since-we-end-up-reading-state-twicesaves-1197-gas-on-average-from-the-tests)
  - [Reading Same state variable twice due to modifier usage(Save 2040 Gas on average from the tests )](#reading-same-state-variable-twice-due-to-modifier-usagesave-2040-gas-on-average-from-the-tests-)
  - [We can save an entire SLOAD(2100 Gas) by short circuiting the operations](#we-can-save-an-entire-sload2100-gas-by-short-circuiting-the-operations)
  - [We can avoid making a function call here by utilizing the short circuit rules](#we-can-avoid-making-a-function-call-here-by-utilizing-the-short-circuit-rules)
  - [Cache function calls](#cache-function-calls)
  - [Unnecessary function call(Saves 369 Gas on average)](#unnecessary-function-callsaves-369-gas-on-average)
  - [Validate function parameters before making function calls or reading any state variables](#validate-function-parameters-before-making-function-calls-or-reading-any-state-variables)
  - [Emit local variables instead of state variable(Save ~100 Gas)](#emit-local-variables-instead-of-state-variablesave-100-gas)
    - [Emit `_maxMintPerBlock` instead of `maxMintPerBlock`](#emit-_maxmintperblock-instead-of-maxmintperblock)
    - [Emit `_maxMintPerBlock` instead of `maxMintPerBlock`](#emit-_maxmintperblock-instead-of-maxmintperblock-1)
    - [Emit `duration` instead of `cooldownDuration`(Save 1 SLOAD)](#emit-duration-instead-of-cooldowndurationsave-1-sload)


## Use constants for variables that don't change(Save a storage SLOT: 2200 Gas)

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L22
```solidity
File: /contracts/StakedUSDeV2.sol
22:  uint24 public MAX_COOLDOWN_DURATION = 90 days;
```
The variable `MAX_COOLDOWN_DURATION` should be declared as a constant variable since it does not change , from the naming , it would seem the intention was to actually make it a constant

```diff
diff --git a/contracts/StakedUSDeV2.sol b/contracts/StakedUSDeV2.sol
index df2bb48..f8fa980 100644
--- a/contracts/StakedUSDeV2.sol
+++ b/contracts/StakedUSDeV2.sol
@@ -19,7 +19,7 @@ contract StakedUSDeV2 is IStakedUSDeCooldown, StakedUSDe {

   USDeSilo public silo;

-  uint24 public MAX_COOLDOWN_DURATION = 90 days;
+  uint24 public constant MAX_COOLDOWN_DURATION = 90 days;
```

## Unnecessary SLOADS inside the constructor(Save 2 SLOADS - 4200 Gas)

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L111-L136
```solidity
File: /contracts/EthenaMinting.sol
111:  constructor(
112:    IUSDe _usde,
113:    address[] memory _assets,
114:    address[] memory _custodians,
115:    address _admin,
116:    uint256 _maxMintPerBlock,
117:    uint256 _maxRedeemPerBlock
118:  ) {
  

134:    // Set the max mint/redeem limits per block
135:    _setMaxMintPerBlock(_maxMintPerBlock);
136:    _setMaxRedeemPerBlock(_maxRedeemPerBlock);
```

In the constructor , we call `_setMaxMintPerBlock()` and `_setMaxRedeemPerBlock()` functions to set `maxMintPerBlock` and `maxRedeemPerBlock` respectively. 
The functions are defined as follows(we only focus on `_setMaxMintPerBlock()`  as they are essentially the same)
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L436-L440

```solidity
436:  function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {
437:    uint256 oldMaxMintPerBlock = maxMintPerBlock;
438:    maxMintPerBlock = _maxMintPerBlock;
439:    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);
440:  }
```

This function will first read the current value of `maxMintPerBlock` and store in a local variable,however when called inside the constructor, we know the value of `maxMintPerBlock` is `0` so we don't necessarily need to cache it inside the constructor, we just need to set it.
Caching inside the constructor would just mean we are using an SLOAD(2100 gas) to cache value `0`. 
I suggest we refactor the constructor as follows which would save us 2 cold SLOADS

```diff
     // Set the max mint/redeem limits per block
-    _setMaxMintPerBlock(_maxMintPerBlock);
-    _setMaxRedeemPerBlock(_maxRedeemPerBlock);
+    maxMintPerBlock = _maxMintPerBlock;
+    maxRedeemPerBlock = _maxRedeemPerBlock;
```

If we really need to emit events inside the constructor, we can define a new event that would emit the current set value ie `_maxMintPerBlock` and `_maxRedeemPerBlock`

## Modifier makes it expensive since we end up reading state twice(Saves 1197 Gas on average from the tests)

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L162-L174

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 4657    | 81745   | 123779 | 195520 |
| After  | 4645    | 80548   | 123639 | 195380 |

```solidity
File: /contracts/EthenaMinting.sol
162:  function mint(Order calldata order, Route calldata route, Signature calldata signature)
163:    external
164:    override
165:    nonReentrant
166:    onlyRole(MINTER_ROLE)
167:    belowMaxMintPerBlock(order.usde_amount)
168:  {
169:    if (order.order_type != OrderType.MINT) revert InvalidOrder();
170:    verifyOrder(order, signature);
171:    if (!verifyRoute(route, order.order_type)) revert InvalidRoute();
172:    if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();
173:    // Add to the minted amount in this block
174:    mintedPerBlock[block.number] += order.usde_amount;
```

The function `mint()` makes use of the modifier `belowMaxMintPerBlock()` which has the following implementation
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L97-L100
```solidity
97:  modifier belowMaxMintPerBlock(uint256 mintAmount) {
98:    if (mintedPerBlock[block.number] + mintAmount > maxMintPerBlock) revert MaxMintPerBlockExceeded();
99:    _;
100:  }
```
Note, the modifier reads `mintedPerBlock[block.number]` which is a state varible.
Our `mint()` function also does the same SLOAD when adding to the minted amount
we can avoid making this two SLOADS by simply in lining this modifier and caching the result of `mintedPerBlock[block.number]` as shown below


```diff
@@ -164,14 +164,15 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
     override
     nonReentrant
     onlyRole(MINTER_ROLE)
-    belowMaxMintPerBlock(order.usde_amount)
   {
+    uint256 _mintedPerBlock = mintedPerBlock[block.number];
+    if (_mintedPerBlock + order.usde_amount > maxMintPerBlock) revert MaxMintPerBlockExceeded();
     if (order.order_type != OrderType.MINT) revert InvalidOrder();
     verifyOrder(order, signature);
     if (!verifyRoute(route, order.order_type)) revert InvalidRoute();
     if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();
     // Add to the minted amount in this block
-    mintedPerBlock[block.number] += order.usde_amount;
+    mintedPerBlock[block.number] = _mintedPerBlock + order.usde_amount;
     _transferCollateral(
       order.collateral_amount, order.collateral_asset, order.benefactor, route.addresses, route.ratios
     );
```
Since the modifier was only being used for the function `mint()` we can go ahead and delete it

## Reading Same state variable twice due to modifier usage(Save 2040 Gas on average from the tests )

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L194-L205
|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 6349    | 40053   | 27127 | 81127 |
| After  | 6337    | 38013   | 20962 | 81017 |

```solidity
File: /contracts/EthenaMinting.sol
194:  function redeem(Order calldata order, Signature calldata signature)
195:    external
196:    override
197:    nonReentrant
198:    onlyRole(REDEEMER_ROLE)
199:    belowMaxRedeemPerBlock(order.usde_amount)
200:  {
201:    if (order.order_type != OrderType.REDEEM) revert InvalidOrder();
202:    verifyOrder(order, signature);
203:    if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();
204:    // Add to the redeemed amount in this block
205:    redeemedPerBlock[block.number] += order.usde_amount;
```
Similar to our previous finding, the function `redeem()` uses the modifier `belowMaxRedeemPerBlock()` which is implemented as below
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L104-L107
```solidity
104:  modifier belowMaxRedeemPerBlock(uint256 redeemAmount) {
105:    if (redeemedPerBlock[block.number] + redeemAmount > maxRedeemPerBlock) revert MaxRedeemPerBlockExceeded();
106:    _;
107:  }
```
We read the state variable `redeemedPerBlock[block.number]` which is also read inside the `redeem()` function. We can inline the modifier which would allow us to cache the call which helps avoid making extra sloads
Refactor the code as shown below.
```diff
@@ -196,13 +191,14 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
     override
     nonReentrant
     onlyRole(REDEEMER_ROLE)
-    belowMaxRedeemPerBlock(order.usde_amount)
   {
+    uint256 _redeemedPerBlock = redeemedPerBlock[block.number];
+    if (_redeemedPerBlock + order.usde_amount > maxRedeemPerBlock) revert MaxRedeemPerBlockExceeded();
     if (order.order_type != OrderType.REDEEM) revert InvalidOrder();
     verifyOrder(order, signature);
     if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();
     // Add to the redeemed amount in this block
-    redeemedPerBlock[block.number] += order.usde_amount;
+    redeemedPerBlock[block.number] = _redeemedPerBlock + order.usde_amount;
     usde.burnFrom(order.benefactor, order.usde_amount);
```

Since the modifier was only being used for the function `redeem()` we can go ahead and delete it

## We can save an entire SLOAD(2100 Gas) by short circuiting the operations

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L413-L433
```solidity
File: /contracts/EthenaMinting.sol
413:  function _transferCollateral(

419:  ) internal {
420:    // cannot mint using unsupported asset or native ETH even if it is supported for redemptions
421:    if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();
```

The if statement has two checks `!_supportedAssets.contains(asset)` and `asset == NATIVE_TOKEN` where the first check involves making a state read while the second check only compares a constant variable to a function parameter.
According to the rules of short circuit , if the first check is true , we do not have to do the second check thus in this case, we should make sure the first check is the cheapest to do.
By reordering as shown below, we can avoid making the state read if `asset == NATIVE_TOKEN` which would save us ~2100 Gas 
```diff
diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol
index 32da3a5..35f4613 100644
--- a/contracts/EthenaMinting.sol
+++ b/contracts/EthenaMinting.sol
@@ -418,7 +418,7 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu
     uint256[] calldata ratios
   ) internal {
     // cannot mint using unsupported asset or native ETH even if it is supported for redemptions
-    if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();
+    if (asset == NATIVE_TOKEN || !_supportedAssets.contains(asset) ) revert UnsupportedAsset();
     IERC20 token = IERC20(asset);
     uint256 totalTransferred = 0;
     for (uint256 i = 0; i < addresses.length; ++i) {
```

## We can avoid making a function call here by utilizing the short circuit rules

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L245-L252
```solidity
File: /contracts/StakedUSDe.sol
245:  function _beforeTokenTransfer(address from, address to, uint256) internal virtual override {
246:    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && to != address(0)) {
247:      revert OperationNotAllowed();
248:    }
249:    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {
250:      revert OperationNotAllowed();
251:    }
252:  }
```

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

## Cache function calls

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L89-L99
```solidity
File: /contracts/StakedUSDe.sol
89:  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
90:    if (getUnvestedAmount() > 0) revert StillVesting();
91:    uint256 newVestingAmount = amount + getUnvestedAmount();
```

Note , we are calling `getUnvestedAmount()` function two times. If we look at the implementation of that function here https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L173-L181
we have the following 
```solidity
173:  function getUnvestedAmount() public view returns (uint256) {
174:    uint256 timeSinceLastDistribution = block.timestamp - lastDistributionTimestamp;

176:    if (timeSinceLastDistribution >= VESTING_PERIOD) {
177:      return 0;
178:    }

180:    return ((VESTING_PERIOD - timeSinceLastDistribution) * vestingAmount) / VESTING_PERIOD;
181:  }
```
Note, in this function, we make two state reads(SLOADS) for `lastDistributionTimestamp` and `vestingAmount`. As this is the first SLOAD in this transaction, this means the variables are COLD thus we use 2100 Gas per variable. Ie 4200 Gas for the two variables read
Calling this function twice would incur a lot of cost(gas). We should cache the results if this call and save them in a local variable as shown below

```diff
diff --git a/contracts/StakedUSDe.sol b/contracts/StakedUSDe.sol
index 0a56a7d..0953b2f 100644
--- a/contracts/StakedUSDe.sol
+++ b/contracts/StakedUSDe.sol
@@ -87,8 +87,9 @@ contract StakedUSDe is SingleAdminAccessControl, ReentrancyGuard, ERC20Permit, E
    * @param amount The amount of rewards to transfer.
    */
   function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
-    if (getUnvestedAmount() > 0) revert StillVesting();
-    uint256 newVestingAmount = amount + getUnvestedAmount();
+    uint256 _unvestedAmount = getUnvestedAmount();
+    if (_unvestedAmount > 0) revert StillVesting();
+    uint256 newVestingAmount = amount + _unvestedAmount;

     vestingAmount = newVestingAmount;
     lastDistributionTimestamp = block.timestamp;
```

**The following finding is somehow related to this one,some confusion on why the devs choose to do the calls** - The next finding will show an alternate optimization

## Unnecessary function call(Saves 369 Gas on average) 

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L89-L99
|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 4359    | 42190   | 44596 | 66916 |
| After  | 4359    | 41821   | 44065 | 66385 |
```solidity
File: /contracts/StakedUSDe.sol
89:  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
90:    if (getUnvestedAmount() > 0) revert StillVesting();
91:    uint256 newVestingAmount = amount + getUnvestedAmount();

93:    vestingAmount = newVestingAmount;
94:    lastDistributionTimestamp = block.timestamp;
95:    // transfer assets from rewarder to this contract
96:    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

98:    emit RewardsReceived(amount, newVestingAmount);
99:  }
```

The function `getUnvestedAmount()` returns `uint256` which means anything greater than or equal to  `0`
we revert if the return is greater than 0, which means the only way we get to execute the function `transferInRewards()` is when `getUnvestedAmount()` returns `0`. This begs the question, why do we call the function on the second line if we can only get there when `getUnvestedAmount() == 0`

```diff
diff --git a/contracts/StakedUSDe.sol b/contracts/StakedUSDe.sol
index 0a56a7d..a378300 100644
--- a/contracts/StakedUSDe.sol
+++ b/contracts/StakedUSDe.sol
@@ -88,7 +88,7 @@ contract StakedUSDe is SingleAdminAccessControl, ReentrancyGuard, ERC20Permit, E
    */
   function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
     if (getUnvestedAmount() > 0) revert StillVesting();
-    uint256 newVestingAmount = amount + getUnvestedAmount();
+    uint256 newVestingAmount = amount;

     vestingAmount = newVestingAmount;
     lastDistributionTimestamp = block.timestamp;
```

## Validate function parameters before making function calls or reading any state variables

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L339-L348
```solidity
File: /contracts/EthenaMinting.sol
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


```diff
diff --git a/contracts/EthenaMinting.sol b/contracts/EthenaMinting.sol
index 32da3a5..cf642d5 100644
--- a/contracts/EthenaMinting.sol
+++ b/contracts/EthenaMinting.sol
@@ -337,13 +337,13 @@ contract EthenaMinting is IEthenaMinting, SingleAdminAccessControl, ReentrancyGu

   /// @notice assert validity of signed order
   function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
-    bytes32 taker_order_hash = hashOrder(order);
-    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
-    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
     if (order.beneficiary == address(0)) revert InvalidAmount();
     if (order.collateral_amount == 0) revert InvalidAmount();
     if (order.usde_amount == 0) revert InvalidAmount();
     if (block.timestamp > order.expiry) revert SignatureExpired();
+    bytes32 taker_order_hash = hashOrder(order);
+    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
+    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
     return (true, taker_order_hash);
   }
```

## Emit local variables instead of state variable(Save ~100 Gas)

### Emit `_maxMintPerBlock` instead of `maxMintPerBlock`

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L436-L440
```solidity
File: /contracts/EthenaMinting.sol
436:  function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {
437:    uint256 oldMaxMintPerBlock = maxMintPerBlock;
438:    maxMintPerBlock = _maxMintPerBlock;
439:    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);
440:  }
```

Since we are setting our state variable `maxRedeemPerBlock` to the function parameter `_maxRedeemPerBlock` , we should emit the function parameter as it is cheaper to read compared to the state variable

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

### Emit `_maxMintPerBlock` instead of `maxMintPerBlock`

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L443-L447
```solidity
File: /contracts/EthenaMinting.sol
443:  function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {
444:    uint256 oldMaxRedeemPerBlock = maxRedeemPerBlock;
445:    maxRedeemPerBlock = _maxRedeemPerBlock;
446:    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);
447:  }
```
Since we are setting our state variable `maxRedeemPerBlock` to the function parameter `_maxRedeemPerBlock` , we should emit the function parameter as it is cheaper to read compared to the state variable
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

### Emit `duration` instead of `cooldownDuration`(Save 1 SLOAD)

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L126-L134
|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 2439    | 3110   | 2439 | 9239 |
| After  | 2425    | 3097   | 2425 | 9225 |

```solidity
File: /contracts/StakedUSDeV2.sol
126:  function setCooldownDuration(uint24 duration) external onlyRole(DEFAULT_ADMIN_ROLE) {
127:    if (duration > MAX_COOLDOWN_DURATION) {
128:      revert InvalidCooldown();
129:    }

131:    uint24 previousDuration = cooldownDuration;
132:    cooldownDuration = duration;
133:    emit CooldownDurationUpdated(previousDuration, cooldownDuration);
134:  }
```

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
```

