## Gas Optimizations

| Number | Issue | Instances |
|-|:-|:-:|
| [[G-01](#g-01-remove-unnecessary-function-call-and-stack-variable-creation)] | Remove unnecessary function call and stack variable creation | 1 |
| [[G-02](#g-02-avoid-emitting-state-variable-unnecessarily)] | Avoid emitting state variable unnecessarily | 1 |
| [[G-03](#g-03-state-variables-can-be-cached-instead-of-re-reading-them-from-storage)] | `State` variables can be cached instead of re-reading them from storage | 1 |
| [[G-04](#g-04-refactor-the-code-to-save-extra-function-calls)] | Refactor the code to save extra function calls | 2 |
| [[G-05](#g-05-swap-if-statements-to-execute-less-gas-consuming-before-more-gas-consuming)] | Swap if() statements to execute less gas consuming before more gas consuming. | 1 |
| [[G-06](#g-06-check-amount-for-0-before-transferring-them)] | Check amount for 0 before transferring them.| 4 |
| [[G-07](#g-07-refactor-if-statements-to-revert-early)] | Refactor if statements to revert early | 3 |
| [[G-08](#g-08-make-state-variables-constant-instead-of-storage-variables-if-its-value-never-changed)] | Make state variables constant instead of storage variables if it's value never changed | 1 |
| [[G-09](#g-09-can-make-the-variable-outside-the-loop-to-save-gas)] | Can Make The Variable Outside The Loop To Save Gas | 1 |




## [G-01] Remove unnecessary function call and stack variable creation 

When `getUnvestedAmount()` returns 0 then it will only pass if condition otherwise revert because return value of  `getUnvestedAmount()`  is uint type . So adding 0 to `amount` is unnecessary and taking it into new variable is also not needed. So use just amount instead of `newVestingAmount` and it will have same working and can save gas of  `getUnvestedAmount()` function call, stack variable creation and 1 add operation.

_1 Instances in 1 File_

```solidity
File : contracts/StakedUSDe.sol

  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
    uint256 newVestingAmount = amount + getUnvestedAmount();
    //@audit if getUnvestedAmount()  return 0 then it will pass if check

    vestingAmount = newVestingAmount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, newVestingAmount);
  }

```
[89-99](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L89C1-L99C4)

```diff
File : contracts/StakedUSDe.sol

  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
-    uint256 newVestingAmount = amount + getUnvestedAmount();

-    vestingAmount = newVestingAmount;
+    vestingAmount = amount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

-    emit RewardsReceived(amount, newVestingAmount);
+    emit RewardsReceived(amount, amount);
  }

```

## [G-02] Avoid emitting state variable unnecessarily.

Use parameter `_maxMintPerBlock` instead of state variable `maxMintPerBlock` because same value assigned in state variable in just above line of just event emit. It saves 1 SLOAD.

_1 Instance in 1 File_

```solidity
File : contracts/EthenaMinting.sol

  function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {
    uint256 oldMaxMintPerBlock = maxMintPerBlock;
    maxMintPerBlock = _maxMintPerBlock;
    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);
  }

```
[436-440](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L436C1-L440C4)

```diff
File : contracts/EthenaMinting.sol

  function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {
     uint256 oldMaxMintPerBlock = maxMintPerBlock;
     maxMintPerBlock = _maxMintPerBlock;
-    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);
+    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, _maxMintPerBlock);
  }

```

```solidity
File : contracts/EthenaMinting.sol

  function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {
    uint256 oldMaxRedeemPerBlock = maxRedeemPerBlock;
    maxRedeemPerBlock = _maxRedeemPerBlock;
    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);
  }

```
[443-447](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L443C1-L447C4)

```diff
File : contracts/EthenaMinting.sol

  function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {
     uint256 oldMaxRedeemPerBlock = maxRedeemPerBlock;
     maxRedeemPerBlock = _maxRedeemPerBlock;
-    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);
+    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, _maxRedeemPerBlock);
  }

```

```solidity
File : contracts/StakedUSDeV2.sol

  function setCooldownDuration(uint24 duration) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (duration > MAX_COOLDOWN_DURATION) {
      revert InvalidCooldown();
    }

    uint24 previousDuration = cooldownDuration;
    cooldownDuration = duration;
    emit CooldownDurationUpdated(previousDuration, cooldownDuration);
  }

```
[126-134](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L126C1-L134C4)

```diff
File : contracts/StakedUSDeV2.sol

  function setCooldownDuration(uint24 duration) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (duration > MAX_COOLDOWN_DURATION) {
      revert InvalidCooldown();
    }

    uint24 previousDuration = cooldownDuration;
    cooldownDuration = duration;
-    emit CooldownDurationUpdated(previousDuration, cooldownDuration);
+    emit CooldownDurationUpdated(previousDuration, duration);
  }

```

## [G-03] `State` variables can be cached instead of re-reading them from storage

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read.

**Note: These instances missed by bot-report**

_1 Instances in 1 File_

Cache `_currentDefaultAdmin`.

```solidity
File : contracts/SingleAdminAccessControl.sol

  function _grantRole(bytes32 role, address account) internal override {
    if (role == DEFAULT_ADMIN_ROLE) {
      emit AdminTransferred(_currentDefaultAdmin, account);
      _revokeRole(DEFAULT_ADMIN_ROLE, _currentDefaultAdmin);
      _currentDefaultAdmin = account;
      delete _pendingDefaultAdmin;
    }
    super._grantRole(role, account);
  }

```
[72-80](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol#L72C1-L80C4)

```diff
File : contracts/SingleAdminAccessControl.sol

  function _grantRole(bytes32 role, address account) internal override {
    if (role == DEFAULT_ADMIN_ROLE) {
+     address current_DefaultAdmin = _currentDefaultAdmin;  
-     emit AdminTransferred(_currentDefaultAdmin, account);
-     _revokeRole(DEFAULT_ADMIN_ROLE, _currentDefaultAdmin);
+     emit AdminTransferred(current_DefaultAdmin, account);
+     _revokeRole(DEFAULT_ADMIN_ROLE, current_DefaultAdmin);
      _currentDefaultAdmin = account;
      delete _pendingDefaultAdmin;
    }
    super._grantRole(role, account);
  }

```

## [G-04] Refactor the code to save extra function calls

It is one liner assignment inside constructor. And previous value is 0 always because first time assigning in constructor so no need to read previous value from state variable and emit them in constructor which is happening inside these functions. It can save 2 function calls and 2 SLOAD  inside them.
_2 Instances in 1 File_

```solidity
File : contracts/EthenaMinting.sol

    _setMaxMintPerBlock(_maxMintPerBlock);
    _setMaxRedeemPerBlock(_maxRedeemPerBlock);
  
```
[135-136](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L135C1-L136C47)

```diff
File : contracts/EthenaMinting.sol

-    _setMaxMintPerBlock(_maxMintPerBlock);
-    _setMaxRedeemPerBlock(_maxRedeemPerBlock);
+    maxMintPerBlock = _maxMintPerBlock;
+    maxRedeemPerBlock = _maxRedeemPerBlock;
  
```

## [G-05] Swap if() statements to execute less gas consuming before more gas consuming.

If 2 if checks have return/revert statements, it is good practice to write those if statements in order less to more gas consuming one. Here later one calculating only one length while it's previous one calculation 2 length so obviously last one is less gas consuming than it's before so we write it it's above.

_1 Instances in 1 File_

```solidity
File : contracts/EthenaMinting.sol

    if (route.addresses.length != route.ratios.length) {
      return false;
    }
    if (route.addresses.length == 0) {
      return false;
    }

```
[357-362](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L357C1-L362C6)

```diff
File : contracts/EthenaMinting.sol

- 357:   if (route.addresses.length != route.ratios.length) {
- 358:    return false;
- 359:  }
  360:  if (route.addresses.length == 0) {
  361:    return false;
  362:   }
+       if (route.addresses.length != route.ratios.length) {
+         return false;
+       }

```

## [G-06] Check amount for 0 before transferring them.

Check amount for 0 when transferring them. If amount is zero it will have no effect and gas will be consumed. And on transferring 0 amount Openzeppelin ERC20 won't revert. It will be successfully executed without no amount transferred and gas will be wasted.

**Note: These instances missed by bot-report**

_4 Instances in 3 Files_

```solidity
File : contracts/EthenaMinting.sol

   function _transferToBeneficiary(address beneficiary, address asset, uint256 amount) internal {
      if (asset == NATIVE_TOKEN) {
        if (address(this).balance < amount) revert InvalidAmount();
        (bool success,) = (beneficiary).call{value: amount}("");
        if (!success) revert TransferFailed();
      } else {
       if (!_supportedAssets.contains(asset)) revert UnsupportedAsset();
       IERC20(asset).safeTransfer(beneficiary, amount);//@audit check amount for 0
       }
    }

```
[401-410](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L401C2-L410C4)

```solidity
File : contracts/StakedUSDe.sol

  function rescueTokens(address token, uint256 amount, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (address(token) == asset()) revert InvalidToken();
    IERC20(token).safeTransfer(to, amount);//@audit check amount for 0
  }

```
[138-141](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L138C1-L141C4)

```solidity
File : contracts/StakedUSDe.sol

  function redistributeLockedAmount(address from, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && !hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {
      uint256 amountToDistribute = balanceOf(from);
      _burn(from, amountToDistribute);//@audit check amountToDistribute for 0
      // to address of address(0) enables burning
      if (to != address(0)) _mint(to, amountToDistribute);

      emit LockedAmountRedistributed(from, to, amountToDistribute);
    } else {
      revert OperationNotAllowed();
    }
  }

```
[148-159](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L148C1-L159C4)

```solidity
File : contracts/StakedUSDeV2.sol

  function unstake(address receiver) external {
    UserCooldown storage userCooldown = cooldowns[msg.sender];
    uint256 assets = userCooldown.underlyingAmount;

    if (block.timestamp >= userCooldown.cooldownEnd) {
      userCooldown.cooldownEnd = 0;
      userCooldown.underlyingAmount = 0;

      silo.withdraw(receiver, assets);//@audit check receiver for address(0) and assets for 0
    } else {
      revert InvalidCooldown();
    }
  }

```
[78-90](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L78C1-L90C4)

## [G-07] Refactor if statements to revert early

If 2 if checks have return/revert statements, it is good practice to write those if statements in order less to more gas consuming one.

**Note: These instances missed by bot-report**

_3 Instance in 1 File_

```solidity
File : contracts/EthenaMinting.sol

  function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
    bytes32 taker_order_hash = hashOrder(order);
    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
    if (order.beneficiary == address(0)) revert InvalidAmount();
    if (order.collateral_amount == 0) revert InvalidAmount();
    if (order.usde_amount == 0) revert InvalidAmount();
    if (block.timestamp > order.expiry) revert SignatureExpired();
    return (true, taker_order_hash);
  }

```
[339-348](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L339C1-L348C4)

```diff
File : contracts/EthenaMinting.sol

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

```solidity
File : contracts/EthenaMinting.sol

364:  if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)

```
[364](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L364C6-L364C121)

```diff
File : contracts/EthenaMinting.sol

-   if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)
+   if (route.addresses[i] == address(0) || route.ratios[i] == 0 || !_custodianAddresses.contains(route.addresses[i]))

```

```solidity
File : contracts/EthenaMinting.sol

 if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();

```
[421](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L421C4-L421C95)

```diff
File : contracts/EthenaMinting.sol

-   if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();
+   if (asset == NATIVE_TOKEN || !_supportedAssets.contains(asset)) revert UnsupportedAsset();

```

## [G-08] Make state variables constant instead of storage variables if it's value never changed 

Declare state variable `MAX_COOLDOWN_DURATION` as constant variable if it's value never changes. Here it will be packed with `silo` which takes 20 bytes so it will not save stirage SLOT heer but it can definitely save some SLOAD (Gcoldsload(2100 GAS) Or Gwarmaccess(100 GAS)) whenever `MAX_COOLDOWN_DURATION` is accessed. It is definitely worth it to make it constant. 

_1 Instance in 1 File_

```diff
File : contracts/StakedUSDeV2.sol

-    uint24 public MAX_COOLDOWN_DURATION = 90 days;
+    uint24 public constant MAX_COOLDOWN_DURATION = 90 days;

```
[22](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L22C2-L22C49)

## [G-09] Can Make The Variable Outside The Loop To Save Gas

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements. By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract

_1 Instance in 1 File_

```solidity
File : contracts/EthenaMinting.sol

    for (uint256 i = 0; i < addresses.length; ++i) {
      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
      totalTransferred += amountToTransfer;
    }

```
[424-428](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L424C1-L428C6)

```diff
File : contracts/EthenaMinting.sol

+   uint256 amountToTransfer
    for (uint256 i = 0; i < addresses.length; ++i) {
-     uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
+     amountToTransfer = (amount * ratios[i]) / 10_000;
      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
      totalTransferred += amountToTransfer;
    }

```
