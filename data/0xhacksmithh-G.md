
### [Gas-01] State Variable could be tightly packed
Both `maxMintPerBlock` and `maxRedeemPerBlock` could be tightly packed inside one storage slot

Gas saved : 2.1k
```diff
  /// @notice max minted USDe allowed per block
- uint256 public maxMintPerBlock; 
  /// @notice max redeemed USDe allowed per block
- uint256 public maxRedeemPerBlock;


+ uint128 public maxMintPerBlock; 

+ uint128 public maxRedeemPerBlock;
```
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L88
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L91
```

### [Gas-02] Instead of calling `_setMaxMintPerBlock()` and `_setMaxRedeemPerBlock()` directly could delete `maxMintPerBlock` and `maxRedeemPerBlock`. So that gas used in extra function jump can be saved
```diff
function disableMintRedeem() external onlyRole(GATEKEEPER_ROLE) { 
-   _setMaxMintPerBlock(0);
-   _setMaxRedeemPerBlock(0);

+   delete maxMintPerBlock;
+   delete maxRedeemPerBlock;
  }
```
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L230-L231
```

### [Gas-03] Before making changes to state variable, you should run pre-existing check(updated value != current value)
```diff
  function removeDelegatedSigner(address _removedSigner) external { 
+   if(delegatedSigner[_removedSigner][msg.sender]){
      delegatedSigner[_removedSigner][msg.sender] = false;
      emit DelegatedSignerRemoved(_removedSigner, msg.sender);
+   }
  }
```
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L241-L244
```


### [Gas-04] Some `memory` variable initialized with default value
```diff
-   uint256 totalRatio = 0;

-   uint256 totalRatio;
```
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L356
```
```diff
-    uint256 totalTransferred = 0;
+    uint256 totalTransferred;
```
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L423
```

### [Gas-05] Crusial checks should present at top
Here both `if` checks should be above of `totalRatio` variable creation
So that if any If checks failed there will be no waste of gas by creation memory variable `totalRatio`
```diff
  function verifyRoute(Route calldata route, OrderType orderType) public view override returns (bool) {
    // routes only used to mint
    if (orderType == OrderType.REDEEM) { 
      return true;
    }
-   uint256 totalRatio = 0; 
    if (route.addresses.length != route.ratios.length) { 
      return false;
    }
    if (route.addresses.length == 0) {
      return false;
    }
+   uint256 totalRatio = 0; 
......
......
```
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L357
```

### [Gas-06] Those Mathematical operation which will not overflow could mark as `unchecked`
```diff
-     totalRatio += route.ratios[i];

+     unchecked{ totalRatio += route.ratios[i]; }
```
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L368
```

### [Gas-07] Low level call can preform with `assembly`
```solidity
  function _transferToBeneficiary(address beneficiary, address asset, uint256 amount) internal { 
    if (asset == NATIVE_TOKEN) {
      if (address(this).balance < amount) revert InvalidAmount();
      (bool success,) = (beneficiary).call{value: amount}(""); 
      if (!success) revert TransferFailed();
```
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L404
```

### [Gas-08] Instead of `> 0` use `!= 0`, which is more gas efficient
`remainingBalance` which is a `uint256` should check against `!= 0`
```diff
-   if (remainingBalance > 0) {

+   if (remainingBalance != 0) {
```
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L423
```


### [Gas-09] `emits` could be more optimizable

```diff
  function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {
-   uint256 oldMaxMintPerBlock = maxMintPerBlock;
-   maxMintPerBlock = _maxMintPerBlock;
-   emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);

+   emit MaxMintPerBlockChanged(maxMintPerBlock, _maxMintPerBlock);
+   maxMintPerBlock = _maxMintPerBlock
  }
```
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L439
```
```diff
  function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {
-   uint256 oldMaxRedeemPerBlock = maxRedeemPerBlock;
-   maxRedeemPerBlock = _maxRedeemPerBlock;
-   emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock); 

+   emit MaxRedeemPerBlockChanged(maxRedeemPerBlock, _maxRedeemPerBlock); 
+   maxRedeemPerBlock = _maxRedeemPerBlock;
  }
```
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L446
```
```diff
  function setCooldownDuration(uint24 duration) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (duration > MAX_COOLDOWN_DURATION) {
      revert InvalidCooldown();
    }

-   uint24 previousDuration = cooldownDuration;
-   cooldownDuration = duration;
-   emit CooldownDurationUpdated(previousDuration, cooldownDuration);

+   emit CooldownDurationUpdated(cooldownDuration, duration);
+   cooldownDuration = duration
```
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L131-L133
```

### [Gas-10] While calculating `newVestingAmount` in `transferInRewards()` no need to add `getUnvestedAmount()` with inputed amount as `getUnvestedAmount() = 0` in that case
As previously it checked that if `getUnvestedAmount()` is > 0 then function revert,
So no need to add that function result in `newVestingAmount`
```diff
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();
-   uint256 newVestingAmount = amount + getUnvestedAmount();

+   uint256 newVestingAmount = amount;
```
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L90-L91
```

### [Gas-11] `MAX_COOLDOWN_DURATION` could be a `constant` state variable
```diff
-   uint24 public MAX_COOLDOWN_DURATION = 90 days;

+   uint24 constant MAX_COOLDOWN_DURATION = 90 days;
```
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L22
```


