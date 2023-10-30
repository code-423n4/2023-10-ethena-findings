## Gas report



#### [G-1] -  Make an important check ahead of others
For a legitimate user, checking whether the order has expired is more important than checking whether the signature is valid. If the time has expired, then the legitimate user will spend extra gas on checking the signature and other information. It is recommended to check for time expiration first, so that if time is up, the law-abiding user will spend less gas
```diff
 function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
+   if (block.timestamp > order.expiry) revert SignatureExpired();
    bytes32 taker_order_hash = hashOrder(order);
    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
    if (order.beneficiary == address(0)) revert InvalidAmount();
    if (order.collateral_amount == 0) revert InvalidAmount();
    if (order.usde_amount == 0) revert InvalidAmount();
-   if (block.timestamp > order.expiry) revert SignatureExpired();
    return (true, taker_order_hash);
  }

```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L339-L346

### [G-2] No need check type of order in verifyRoute()
Because, this function called only in mint() function and in mint() function there is check ` if (order.order_type != OrderType.MINT) revert InvalidOrder();`. Delete paramter - order type.
```diff
//EthenaMinting.sol
 function mint(Order calldata order, Route calldata route, Signature calldata signature)
    external
    override
    nonReentrant
    onlyRole(MINTER_ROLE)
    belowMaxMintPerBlock(order.usde_amount)
  { 
    if (order.order_type != OrderType.MINT) revert InvalidOrder(); // <--- 
    verifyOrder(order, signature);
-   if (!verifyRoute(route, order.order_type)) revert InvalidRoute();
+	if (!verifyRoute(route)) revert InvalidRoute();
	...
 }
 
 
-  function verifyRoute(Route calldata route, OrderType orderType) public view override returns (bool) {
+  function verifyRoute(Route calldata route) public view override returns (bool) {
-		// routes only used to mint
-    	if (orderType == OrderType.REDEEM) {
-      		return true;
-    	}
-    ...

	}

// interfaces/IEthenaMinting.sol
	...
-	function verifyRoute(Route calldata route, OrderType order_type) external view returns (bool);
+	function verifyRoute(Route calldata route) external view returns (bool);
 	 ...

```

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L162-L171
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L351-L374
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/interfaces/IEthenaMinting.sol#L63

### [G-3] Combine 2 loops into 1
if the verifyRoute() function is called only from mint(), then we can combine 2 loops: the cycle from verifyRoute with the cycle from _transferCollateral
```diff
// EthenaMinting.sol
 function verifyRoute(Route calldata route, OrderType orderType) public view override returns (bool) {
...
-    uint256 totalRatio = 0;
-    for (uint256 i = 0; i < route.addresses.length; ++i) {
-      if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)
-      {
-        return false;
-      }
-      totalRatio += route.ratios[i];
-    }
-    if (totalRatio != 10_000) {
-      return false;
-    }
  ...
}


+ error InvalidRatio();
  function _transferCollateral(
    uint256 amount,
    address asset,
    address benefactor,
    address[] calldata addresses,
    uint256[] calldata ratios
  ) internal {
    // cannot mint using unsupported asset or native ETH even if it is supported for redemptions
    if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();
    IERC20 token = IERC20(asset);
    uint256 totalTransferred = 0;
+   uint256 totalRatio = 0;
    for (uint256 i = 0; i < addresses.length; ++i) {
+     if (!_custodianAddresses.contains(addresses[i]) || addresses[i] == address(0) || ratios[i] == 0)
+     {
+       revert InvalidCustodianAddress();
+     }
+     totalRatio += ratios[i];
      
      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
      totalTransferred += amountToTransfer;
    }
    
+    if (totalRatio != 10_000) {
+     revert InvalidRatio();
+   }
    
    uint256 remainingBalance = amount - totalTransferred;
    if (remainingBalance > 0) {
      token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);
    }
  }

```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L351-L374
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L351-L374

### [G-3] Unnecessary reading of storage variable in _setMaxMintPerBlock()
No need to create a memory variable and read the storage variable 2 times
```diff
  function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {
-    uint256 oldMaxMintPerBlock = maxMintPerBlock;
-    maxMintPerBlock = _maxMintPerBlock;
-    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);
+	 emit MaxMintPerBlockChanged(maxMintPerBlock, _maxMintPerBlock);
+    maxMintPerBlock = _maxMintPerBlock;
  }
```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L436-L440

### [G-4] Unnecessary reading of storage variable in _setMaxRedeemPerBlock()
No need to create a memory variable and read the storage variable 2 times
```diff
  function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {
-    uint256 oldMaxRedeemPerBlock = maxRedeemPerBlock;
-    maxRedeemPerBlock = _maxRedeemPerBlock;
-    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);
+    emit MaxRedeemPerBlockChanged(maxRedeemPerBlock, _maxRedeemPerBlock);
+    maxRedeemPerBlock = _maxRedeemPerBlock;
  }
```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L443-L446

### [G-5] Unnecessary call getUnvestedAmount()
A user with the REWARDER_ROLE role will not be able to deposit funds into the contract if the previous period has not ended. This is checked in the first line. Therefore, there is no need to use the concepts old meaning and new. We always have the old value equal to 0, and the new one does not add up to the old one.
```diff
// StakedUSDe.sol
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting(); // <----------
-   uint256 newVestingAmount = amount + getUnvestedAmount();   // amount + 0
-   vestingAmount = newVestingAmount;
+	vestingAmount = amount;   
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

-   emit RewardsReceived(amount, newVestingAmount);
+   emit RewardsReceived(amount);

// interfaces/IStakedUSDe.sol
-  event RewardsReceived(uint256 indexed amount, uint256 newVestingUSDeAmount);
+  event RewardsReceived(uint256 newVestingUSDeAmount);
```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L89-L99

### [G-6] Unnecessary check role
Checking whether the fund recipient has a role `FULL_RESTRICTED_STAKER_ROLE `also occurs in the _beforeTokenTransfer function, Read comments(natspec) of function _beforeTokenTranfer. It tells about - "This include minting and burning". So you can delete it here. 
   * minting and burning. Disables transfers
```diff
 function redistributeLockedAmount(address from, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {
-   if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && !hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {
+	if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from)) {
      uint256 amountToDistribute = balanceOf(from);
      _burn(from, amountToDistribute);
      // to address of address(0) enables burning
      if (to != address(0)) _mint(to, amountToDistribute);

      emit LockedAmountRedistributed(from, to, amountToDistribute);
    } else {
      revert OperationNotAllowed();
    }
  }
  
/**
   * @dev Hook that is called before any transfer of tokens. This includes
   * minting and burning. Disables transfers from or to of addresses with the FULL_RESTRICTED_STAKER_ROLE role.
   */

  function _beforeTokenTransfer(address from, address to, uint256) internal virtual override {  
```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L148C2-L159C4

### [G-7] Dont use else
In this issue, I'll use code from G-6 (after)
```diff
 function redistributeLockedAmount(address from, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {
-	if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from)) {
+	if (!hasRole(FULL_RESTRICTED_STAKER_ROLE, from)) revert OperationNotAllowed();
      uint256 amountToDistribute = balanceOf(from);
      _burn(from, amountToDistribute);
      // to address of address(0) enables burning
      if (to != address(0)) _mint(to, amountToDistribute);

      emit LockedAmountRedistributed(from, to, amountToDistribute);
-    } else {
-      revert OperationNotAllowed();
-    }
  }
```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L148C2-L159C4

### [G-8] Unnecessary check, that receiver is not zero address
Checking if the recipient of funds is not null address is already in the mint function in the OZ library.
```diff
  function redistributeLockedAmount(address from, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && !hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {
      uint256 amountToDistribute = balanceOf(from);
      _burn(from, amountToDistribute);
      // to address of address(0) enables burning
-     if (to != address(0)) _mint(to, amountToDistribute);
+	  _mint(to, amountToDistribute);

      emit LockedAmountRedistributed(from, to, amountToDistribute);
    } else {
      revert OperationNotAllowed();
    }
  }

// openzeppelin/contracts/token/ERC20.sol
    function _mint(address account, uint256 value) internal {
        if (account == address(0)) {
            revert ERC20InvalidReceiver(address(0));
        }
        _update(address(0), account, value);
    }
```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L148C2-L159C4
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/94697be8a3f0dfcd95dfb13ffbd39b5973f5c65d/contracts/token/ERC20/ERC20.sol#L226-L228

### [G-9] Unnecessary memory variable
You can not create a memory variable, but use the value directly as a function argument.
```diff

  function unstake(address receiver) external {
    UserCooldown storage userCooldown = cooldowns[msg.sender];
-   uint256 assets = userCooldown.underlyingAmount;

    if (block.timestamp >= userCooldown.cooldownEnd) {
      userCooldown.cooldownEnd = 0;
      userCooldown.underlyingAmount = 0;

-     silo.withdraw(receiver, assets);
+	  silo.withdraw(receiver, userCooldown.underlyingAmount);
    } else {
      revert InvalidCooldown();
    }
  }
```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L78C1-L90C4

### [G-10] No need write zero value to variable
Overwriting a non-zero value is cheaper than writing 0 to a variable, and then again a new value. New value will be just assigned in other functions (cooldownAssets / cooldownShares)
```diff
  function unstake(address receiver) external {
    UserCooldown storage userCooldown = cooldowns[msg.sender];
    uint256 assets = userCooldown.underlyingAmount;

    if (block.timestamp >= userCooldown.cooldownEnd) {
-     userCooldown.cooldownEnd = 0;
      userCooldown.underlyingAmount = 0;

      silo.withdraw(receiver, assets);
    } else {
      revert InvalidCooldown();
    }
  }
```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L78C1-L90C4

### [G-11] Dont use else
```diff
  function unstake(address receiver) external {
    UserCooldown storage userCooldown = cooldowns[msg.sender];
    uint256 assets = userCooldown.underlyingAmount;

-   if (block.timestamp >= userCooldown.cooldownEnd) {
+	if (block.timestamp < userCooldown.cooldownEnd) revert InvalidCooldown();
      userCooldown.cooldownEnd = 0;
      userCooldown.underlyingAmount = 0;

      silo.withdraw(receiver, assets);
-    } else {
-      revert InvalidCooldown();
-    }
  }
```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L78C1-L90C4

### [G-12] Unnecessary memory variable
You can not create a memory variable, but use the value directly as a function argument.
```diff
  function setCooldownDuration(uint24 duration) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (duration > MAX_COOLDOWN_DURATION) {
      revert InvalidCooldown();
    }

-   uint24 previousDuration = cooldownDuration;
-   cooldownDuration = duration;
-   emit CooldownDurationUpdated(previousDuration, cooldownDuration);
+   emit CooldownDurationUpdated(cooldownDuration, duration);
+   cooldownDuration = duration;
  }
```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L126-L134

### [G-13] No need use uint104 in struct
It makes no sense use type uint104 for the first variable in the structure. Due to the fact that the type is not uint256, additional gas is spent on conversion. The data in the structure still takes up 2 slots.
```diff
// interfaces/IStakedUSDeCooldown.sol
struct UserCooldown {
- uint104 cooldownEnd;
+ uint256 cooldownEnd;
  uint256 underlyingAmount;
}

```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/interfaces/IStakedUSDeCooldown.sol#L7-L10

### [G-14] Use msg.sender instead variable
The Silo contract is always deployed by the StakedUSDeV2 contract. And the address of the StakedUSDeV2 contract is passed in the constructor. You don’t have to waste gas on the variable, but use the msg.sender value in the silo contract to write the address of the StakedUSDeV2 contract.
```diff
// USDeSilo.sol
-  constructor(address stakingVault, address usde) {
+  constructor(address usde) {
-    STAKING_VAULT = stakingVault;
+    STAKING_VAULT = msg.sender;
     USDE = IERC20(usde);
  }
  
// StakedUSDeV2.sol
constructor(IERC20 _asset, address initialRewarder, address owner) StakedUSDe(_asset, initialRewarder, owner) {
-   silo = new USDeSilo(address(this), address(_asset));
+   silo = new USDeSilo(address(_asset));
    cooldownDuration = MAX_COOLDOWN_DURATION;
  }
```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L42-L44
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDeSilo.sol#L18-L21