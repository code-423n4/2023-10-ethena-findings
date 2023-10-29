# [G-01] Checking for revert in  `unstake()` in `StakedUSDeV2.sol` can be moved up

[File: StakedUSDeV2.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L78-L90)
```
function unstake(address receiver) external {
    UserCooldown storage userCooldown = cooldowns[msg.sender];
    uint256 assets = userCooldown.underlyingAmount;

    if (block.timestamp >= userCooldown.cooldownEnd) {
      userCooldown.cooldownEnd = 0;
      userCooldown.underlyingAmount = 0;

      silo.withdraw(receiver, assets);
    } else {
      revert InvalidCooldown();
    }
  }
```

There's no need to get `uint256 assets = userCooldown.underlyingAmount;` before checking if `block.timestamp >= userCooldown.cooldownEnd`.

Please consider changing above code-snippet to:

```
function unstake(address receiver) external {
    UserCooldown storage userCooldown = cooldowns[msg.sender];
   
    if (block.timestamp >= userCooldown.cooldownEnd) {
      uint256 assets = userCooldown.underlyingAmount;
      userCooldown.cooldownEnd = 0;
      userCooldown.underlyingAmount = 0;

      silo.withdraw(receiver, assets);
    } else {
      revert InvalidCooldown();
    }
  }
```

# [G-02] Check if `amount` is non-zero before performing `transfer()`

[File: USDeSilo.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDeSilo.sol#L28)
```
function withdraw(address to, uint256 amount) external onlyStakingVault {
    USDE.transfer(to, amount);
  }
```

When `amount == 0`, then calling `transfer()` is a waste of gas - thus literally nothing will be transfered. It's a good practice to check if `amount > 0` before performing `transfer()`.


# [G-03] `renounceRole()` can be optimized

[File: SingleAdminAccessControl.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/SingleAdminAccessControl.sol#L58-L60)
```
  function renounceRole(bytes32 role, address account) public virtual override notAdmin(role) {
    super.renounceRole(role, account);
  }
```

The current implementation calls OpenZeppelins `renounceRole()`, which checks if caller is `_msgSender()`:

```
    function renounceRole(bytes32 role, address callerConfirmation) public virtual {
        if (callerConfirmation != _msgSender()) {
            revert AccessControlBadConfirmation();
        }

        _revokeRole(role, callerConfirmation);
    }
```

In the current context, this check is not needed - we can call `_revokeRole` directly on `msg.sender`:

```
  function renounceRole(bytes32 role, address account) public virtual override notAdmin(role) {
     _revokeRole(role, msg.sender);
  }
```

OpenZeppelin's implementation, implements additional safeguard - so that `renounceRole()` won't be called by accident. We need to provide our own address (`msg.sender`) as `callerConfirmation` and then call this function. If provided `callerConfirmation` matches the caller (`msg.sender`) - role will be renounce. However, if we want to save some gas - we can remove that safeguard. In our proposed fix - anyone who calls `renounceRole()`, will renounce role for himself/herself.


# [G-04] `uint104(block.timestamp) + cooldownDuration` can be unchecked

Since `cooldownDuration` cannot exceed `MAX_COOLDOWN_DURATION` which is defined as `90 days` and  `block.timestamp` will be greater than `type(uint104).max` after 22 September 2612 - it's reasonable to assume, that 
`uint104(block.timestamp) + cooldownDuration` won't overflow and can be unchecked.

`MAX_COOLDOWN_DURATION = 90 days = 7776000`
`type(uint104).max = 20282409603651670423947251286015`

`timestamp(20282409603651670423947251286015 - 7776000) = 22 September 2612 02:40:03`. This implies, that this addition won't overflow till 22 September 2612 - so it's reasonable to make it unchecked.

Below lines can be unchecked:

[File: StakedUSDeV2.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L100)
```
    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
```

[File: StakedUSDeV2.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L116)
```
    cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
```

# [G-05] Change the order of emitting event - to avoid declaring additional variable in `StakedUSDeV2.sol`

[File: StakedUSDeV2.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L131-L133)
```
uint24 previousDuration = cooldownDuration;
cooldownDuration = duration;
emit CooldownDurationUpdated(previousDuration, cooldownDuration);
```

You can emit `CooldownDurationUpdated` event before updating `cooldownDuration`. That way, there will be no need to declare `previousDuration`:

```
emit CooldownDurationUpdated(cooldownDuration, duration);
cooldownDuration = duration;
```


# [G-06] Function `transferInRewards()` from `StakedUSDe.sol` can be optimized to avoid calling `getUnvestedAmount()` twice.

Calling the same function is a waste of gas. It's much better idea to call it once and save the result to local variable.
Since we've already declared `newVestingAmount` variable - we can utilize it to store `getUnvestedAmount()` there:

```
function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    
    uint256 newVestingAmount = getUnvestedAmount();
    if (newVestingAmount > 0) revert StillVesting();
    newVestingAmount = newVestingAmount + amount;

    vestingAmount = newVestingAmount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, newVestingAmount);
  }
```

# [G-07] Unnecessary variable declaration in `StakedUSDe.sol`

In both functions: `addToBlacklist()` and `removeFromBlacklist()`, there's no need to declare variable `bytes32 role`. This variable is being used only once, which means it doesn't need to be declared at all.

Code can be changed to:

```
 function addToBlacklist(address target, bool isFullBlacklisting)
    external
    onlyRole(BLACKLIST_MANAGER_ROLE)
    notOwner(target)
  {
   
    _grantRole(isFullBlacklisting ? FULL_RESTRICTED_STAKER_ROLE : SOFT_RESTRICTED_STAKER_ROLE, target);
  }
```

```
 function removeFromBlacklist(address target, bool isFullBlacklisting)
    external
    onlyRole(BLACKLIST_MANAGER_ROLE)
    notOwner(target)
  {
    _revokeRole(isFullBlacklisting ? FULL_RESTRICTED_STAKER_ROLE : SOFT_RESTRICTED_STAKER_ROLE, target);
  }
```


# [G-08] Redundant address(0) check in `redistributeLockedAmount()` in `StakedUSDe.sol`

[File: StakedUSDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L153)
```
if (to != address(0)) _mint(to, amountToDistribute);
```

OpenZeppelin's `_mint()` already checks for address(0), thus checking it for the second time in `StakedUSDe.sol` is not needed


# [G-09] Move condition which uses less gas on the left of `AND` operator in `StakedUSDe.sol`

[File: StakedUSDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L246)
```
if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && to != address(0)) {
```

`A && B` condition is true only, when both `A` and `B` conditions evaluate to `true`. This means, that when `A` is `false`, then Solidity does not evaluate `B` (since no matter what `B` is, `A & B` is false).
This behavior implies, that `A` will be evaluated more times than `B` (`B` will be evaluated only when `A = true`). In that case, it's much more efficient to make sure that `A` uses less gas than `B`.

`to != address(0)` costs less gas than `hasRole(FULL_RESTRICTED_STAKER_ROLE, from)`, thus order of conditions should be changed:

```
if (to != address(0) && hasRole(FULL_RESTRICTED_STAKER_ROLE, from)) {
```


# [G-10] `transferToCustody()` in `EthenaMinting.sol` does not verify if `amount > 0`

[File: EhtenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L247-L256)
```
  function transferToCustody(address wallet, address asset, uint256 amount) external nonReentrant onlyRole(MINTER_ROLE) {
    if (wallet == address(0) || !_custodianAddresses.contains(wallet)) revert InvalidAddress();
    if (asset == NATIVE_TOKEN) {
      (bool success,) = wallet.call{value: amount}("");
      if (!success) revert TransferFailed();
    } else {
      IERC20(asset).safeTransfer(wallet, amount);
    }
    emit CustodyTransfer(wallet, asset, amount);
  }
```

It's a waste of gas to perform `wallet.call{value: 0}("")` or `IERC20(asset).safeTransfer(wallet, 0)` operations. If `amount` is 0, nothing will be transfered.
Perform additional check `if (amount !=0)` before calling either `wallet.call{value: 0}("")` or `IERC20(asset).safeTransfer(wallet, 0)`.


# [G-11] `_assets.length` is calculated twice in `EthenaMinting.sol`

[File: EhtenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L120-L226)
```
 if (_assets.length == 0) revert NoAssetsProvided();
    if (_admin == address(0)) revert InvalidZeroAddress();
    usde = _usde;

    _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);

    for (uint256 i = 0; i < _assets.length; i++) {
```

Calculating the length of array may take a lot of gas. It's always a good idea to calculate it once and save the result to local variable.
`_assets.length` is calculated firstly at line 120, and then - 2ndly - at line 126.
Please notice, that the bot-race reported only that `_assets.length` is being used inside a loop. While it's true, it's worth to notice, that it's used also at line 120. This means, that `_assets.length` can be cached before line 120.



# [G-12] In `verifyRouter()` from `EthenaMinting.sol` - variable `totalRatio` can be declared later

Before `totalRatio` is being used, function performs two additional checks: `if (route.addresses.length != route.ratios.length)` and  `if (route.addresses.length == 0)`. If any of them evalutes to `true` - functions will return with `false` without a need of using `totalRatio`. This means, that `uint256 totalRatio = 0;` can be declared later, just before the loop at line 363:

```
uint256 totalRatio = 0;
for (uint256 i = 0; i < route.addresses.length; ++i) {

```



# [G-13] Change the order of emitting event - to avoid declaring additional variable in `EtheneMinting.sol` 

[File: EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L436-L447)
```
  function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {
    uint256 oldMaxMintPerBlock = maxMintPerBlock;
    maxMintPerBlock = _maxMintPerBlock;
    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);
  }

  /// @notice Sets the max redeemPerBlock limit
  function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {
    uint256 oldMaxRedeemPerBlock = maxRedeemPerBlock;
    maxRedeemPerBlock = _maxRedeemPerBlock;
    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);
  }
```

Variables `oldMaxMintPerBlock` and `oldMaxRedeemPerBlock` are used only once, which means, that they do not need to be declared at all. Getting rid of these variables requires to emit events a little earlier, like this:

```
  function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {
    emit MaxMintPerBlockChanged(maxMintPerBlock, _maxMintPerBlock);
    maxMintPerBlock = _maxMintPerBlock;
  }

  /// @notice Sets the max redeemPerBlock limit
  function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {
    emit MaxRedeemPerBlockChanged(maxRedeemPerBlock, _maxRedeemPerBlock);
    maxRedeemPerBlock = _maxRedeemPerBlock; 
  }
```


# [G-14] `verifyOrder()` from `EthenaMinting.sol` can be optimized by performing signature verification later

Calculating `hashOrder()` and `ECDSA.recover()` is the most gas-costly operation in the whole `verifyOrder()` function.
In the current implementation - we firstly check the `ECDSA.recover()`, and then, we perform additional validations (like if `order.expiry` hasn't expired, if `order.usde_ammount` is not zero, etc.).
Basically, if any of the `Order` parameters will be invalid - function will revert. This implies, that it will be more effective to firstly validate those parameters - and then - when they are valid - verify `ESDSA` signature. In that case - if any of `Order` fields won't be valid - we won't spend additional gas on the signature verification.
Here's the proposed fix:

```
  function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
    
    if (order.beneficiary == address(0)) revert InvalidAmount();
    if (order.collateral_amount == 0) revert InvalidAmount();
    if (order.usde_amount == 0) revert InvalidAmount();
    if (block.timestamp > order.expiry) revert SignatureExpired();

    // @audit: we are sure that Order is valid, now we can verify if signature is correct. If Order was invalid, we had reverted before using gas on hashOrder() and ECDSA.recover()
   
    bytes32 taker_order_hash = hashOrder(order);
    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
    return (true, taker_order_hash);
  }
```


# [G-15] Unnecessary variable declaration in `_deduplicateOrder()`, `verifyNonce()` in `EthenaMinting.sol`

Variable `invalidatorStorage ` is used only once, which means, it does not need to be declared at all. Below code:

[File: EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L391-395)
```
 function _deduplicateOrder(address sender, uint256 nonce) private returns (bool) {
    (bool valid, uint256 invalidatorSlot, uint256 invalidator, uint256 invalidatorBit) = verifyNonce(sender, nonce);
    mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
    invalidatorStorage[invalidatorSlot] = invalidator | invalidatorBit;
    return valid;
  }
```

can be changed to:

```
 function _deduplicateOrder(address sender, uint256 nonce) private returns (bool) {
    (bool valid, uint256 invalidatorSlot, uint256 invalidator, uint256 invalidatorBit) = verifyNonce(sender, nonce);
    _orderBitmaps[sender][invalidatorSlot] = invalidator | invalidatorBit;
    return valid;
  }
```

To prove that it will save gas, a quick test in Remix IDE has been performed:

```
 function _deduplicateOrderAAA(address sender, uint256 nonce) public returns (bool) {
    uint gas = gasleft();
    
    (bool valid, uint256 invalidatorSlot, uint256 invalidator, uint256 invalidatorBit) = verifyNonce(sender, nonce);
    mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
    invalidatorStorage[invalidatorSlot] = invalidator | invalidatorBit;
    console.log(gas - gasleft());
    return valid;
  }

   function _deduplicateOrderBBB(address sender, uint256 nonce) public returns (bool) {
    uint gas = gasleft();
    
    (bool valid, uint256 invalidatorSlot, uint256 invalidator, uint256 invalidatorBit) = verifyNonce(sender, nonce);
    _orderBitmaps[sender][invalidatorSlot] = invalidator | invalidatorBit;
    console.log(gas - gasleft());
    return valid;
  }
```

The results look as below:
```
_deduplicateOrderAAA(1, 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4) -> 22742 gas used
_deduplicateOrderAAA(2, 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4) -> 5642 gas used


_deduplicateOrderBBB(1, 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4) -> 22731 gas used
_deduplicateOrderBBB(2, 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4) -> 5631 gas used
```

This implies, that proposed fix will save some gas.

The same issue occurs in `verifyNonce()`
Below code-base:

[File: EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L377-L386)
```
  function verifyNonce(address sender, uint256 nonce) public view override returns (bool, uint256, uint256, uint256) {
    if (nonce == 0) revert InvalidNonce();
    uint256 invalidatorSlot = uint64(nonce) >> 8;
    uint256 invalidatorBit = 1 << uint8(nonce);
    mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
    uint256 invalidator = invalidatorStorage[invalidatorSlot];
    if (invalidator & invalidatorBit != 0) revert InvalidNonce();

    return (true, invalidatorSlot, invalidator, invalidatorBit);
  }
```

can be changed to:

```
 function verifyNonce(address sender, uint256 nonce) public view override returns (bool, uint256, uint256, uint256) {
    if (nonce == 0) revert InvalidNonce();
    uint256 invalidatorSlot = uint64(nonce) >> 8;
    uint256 invalidatorBit = 1 << uint8(nonce);
    uint256 invalidator = _orderBitmaps[sender][invalidatorSlot];
    if (invalidator & invalidatorBit != 0) revert InvalidNonce();

    return (true, invalidatorSlot, invalidator, invalidatorBit);
  }
```


# [G-16] There's no need to check for `route.addresses[i] == address(0)` in `verifyRoute()` in `EthenaMinting.sol`

[File: EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L363-L365)
```
 for (uint256 i = 0; i < route.addresses.length; ++i) {
      if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)
```

There are multiple of checks in this conditions. However, they can be reduced.

When we take a look at function `addCustodianAddress()`, it looks like this:

[File: EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L298-L303)
```
  function addCustodianAddress(address custodian) public onlyRole(DEFAULT_ADMIN_ROLE) {
    if (custodian == address(0) || custodian == address(usde) || !_custodianAddresses.add(custodian)) {
      revert InvalidCustodianAddress();
    }
    emit CustodianAddressAdded(custodian);
  }
```

Function `addCustodianAddress()` cannot add custodian, when custodian is `address(0)`. Whenever we try to call: `addCustodianAddress(address(0))`, function will revert with `InvalidCustodianAddress()`.
This implies, that when `route.addresses[i] == address(0)`, then `!_custodianAddresses.contains(route.addresses[i])` will return `true` (it's impossible to add `address(0)` to supported custodians list:

```
1. It's impossbile to add address(0) to custodians list
2. _custodianAddresses cannot contain address(0), because it's impossible to add address(0)
3. _custodianAddresses.contains(address(0)) returns false, because we know that custodian list does not contain address(0), since it's impossible to add that address to custodian list
4. !_custodianAddresses.contains(address(0)) returns true (since ! is negation)
```

Since `!_custodianAddresses.contains(route.addresses[i])` already told as that `route.addresses[i]` is not `address(0)`, another check `route.addresses[i] == address(0)` is redundant.
This implies, that we can simplify above condition, from: `if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)` to `if (!_custodianAddresses.contains(route.addresses[i]) || route.ratios[i] == 0)`.


# [G-17] `route.addresses.length` is calculated multiple of times in `verifyRoute()` in `EthenaMinting.sol`

[File: EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L357-L363)
```
    if (route.addresses.length != route.ratios.length) {
      return false;
    }
    if (route.addresses.length == 0) {
      return false;
    }
    for (uint256 i = 0; i < route.addresses.length; ++i) {
```

Calculating the length of array may take a lot of gas. It's always a good idea to calculate it once and save the result to local variable.
`route.addresses.length` is calculated firstly at line 357, and then - secondly - at line 360, then - for the 3rd time - at line 363
Please notice, that the bot-race reported only that `route.addresses.length` is being used inside a loop. While it's true, it's worth to notice, that it's used also at line 357 and at line 360. This means, that `route.addresses.length` can be cached:

```
uint256 routeLength = route.addresses.length;
if (routeLength != route.ratios.length) {
      return false;
    }
    if (routeLength == 0) {
      return false;
    }
    for (uint256 i = 0; i < routeLength; ++i) {
```

# [G-18] `_transferCollateral()` calls `safeTransferFrom()` even when transferred amount is 0

[File: EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L424-L428)
```
for (uint256 i = 0; i < addresses.length; ++i) {
      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
      totalTransferred += amountToTransfer;
    }

```

When `amountToTransfer` is 0, performing `token.safeTransferFrom` is not needed. It's a waste of gas to perform 0 transfer. We can check if `amountToTransfer` is greater than 0, before performing transfer:

```
for (uint256 i = 0; i < addresses.length; ++i) {
      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
      if (amountToTransfer != 0) {
          token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
          totalTransferred += amountToTransfer;
        }
    }
```

That way, when `amountToTransfer` is 0, we save gas, because we won't perform two redundant operations:
* `token.safeTransferFrom(benefactor, addresses[i], 0)` - transferring 0 does not affect token's balance, thus it's redundant
* `totalTransferred += 0` - adding 0 does not change the value of `totalTransferred`


# [G-19] Struct `Order` can be packed into fewer storage slots

Examining `ORDER_TYPE` constant allows us to establish how struct `Order` looks like and how many storage slots it uses:

[File: EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L35-L37)
```
bytes32 private constant ORDER_TYPE = keccak256(
    "Order(uint8 order_type,uint256 expiry,uint256 nonce,address benefactor,address beneficiary,address collateral_asset,uint256 collateral_amount,uint256 usde_amount)"
  );
```

This can be confirmed by looking at `interfaces/IEthenaMinting.sol` - however, please notice, that this file is out of scope:

```
 struct Order {
    OrderType order_type;
    uint256 expiry;
    uint256 nonce;
    address benefactor;
    address beneficiary;
    address collateral_asset;
    uint256 collateral_amount;
    uint256 usde_amount;
  }
```

The current ordering of `Order`'s fields uses 8 storage slots:

```
Order(uint8 order_type,uint256 expiry,uint256 nonce,address benefactor,address beneficiary,address collateral_asset,uint256 collateral_amount,uint256 usde_amount)
_____|SLOT 0__________|SLOT 1________|SLOT 2_______|SLOT 3____________|SLOT 4_____________|SLOT 5__________________|SLOT 6___________________|SLOT 7_____________|
```

However, we can change the `Order` to:

```
 struct Order {
    OrderType order_type;
    address benefactor;
    address beneficiary;
    uint256 expiry;
    uint256 nonce;
    address collateral_asset;
    uint256 collateral_amount;
    uint256 usde_amount;
  }
```

That way, we will use 7 slots:

```
Order(uint8 order_type,address benefactor,address beneficiary,uint256 expiry,uint256 nonce,address collateral_asset,uint256 collateral_amount,uint256 usde_amount)
_____|SLOT 0_____________________________|SLOT 1_____________|SLOT 2________|SLOT 3_______|SLOT 4__________________|SLOT 5___________________|SLOT 6_____________|
```


# [G-20] Struct `Order` can be packed into fewer storage slots by changing the type of `expiry` and `nonce`

This issue is related to G-19 optimization. It turns out, that we can reduce the number of `Order`'s storage slots even more.

Since `expiry` is always compared to `block.timestamp`:

[File: EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L346)
```
if (block.timestamp > order.expiry) revert SignatureExpired();
```

We can represent it as `uint64` instead of `uint256`. The max value of `uint64` is 2**64 - 1, which, after converting to UNIX timestamp is `Jul 21 2554 23:34:33`. `expiry` won't overflow till Jun 21 2554, when it will be represented as `uint64`.

Additionally, when we look at the nonce implementation:

[File: EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L379)
```
 uint256 invalidatorSlot = uint64(nonce) >> 8;
```

we see, it's already casted to `uint64`, thus it can be represented in `uint64`.

This leads us to the below, updated struct:

```
 struct Order {
    OrderType order_type;
    address benefactor;
    address beneficiary;
    uint64 expiry;
    uint64 nonce;
    address collateral_asset;
    uint256 collateral_amount;
    uint256 usde_amount;
  }
```

This implies, that we can reduce the number of storage slots to 5:

```
Order(uint8 order_type,address benefactor,address beneficiary,uint64 expiry,uint64 nonce,address collateral_asset,uint256 collateral_amount,uint256 usde_amount)
_____|SLOT 0_____________________________|SLOT 1___________________________|SLOT 2_______________________________|SLOT 3___________________|SLOT 4_____________|
```


# [G-21] Subtraction in `_transferCollateral()` in `EthenaMinting.sol` can be unchecked

[File: EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L431)
```
token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);
```

`addresses.length - 1` won't overflow, thus it can be unchecked.

Please notice, that `_transferCollateral()` is called only in function `mint()`. Before that call, `verifyRoute()` is being called:

```
if (!verifyRoute(route, order.order_type)) revert InvalidRoute();
```

When we take a look at `verifyRoute()` implementation, we can see, that it requires that `route.addresses.length` is not zero.
Since `route.addresses` is non-zero, we can be sure, that `addresses.length - 1` won't underflow (`addresses.length` has to be >= 1).
This implies, that this subtraction can be unchecked, since it won't overflow.