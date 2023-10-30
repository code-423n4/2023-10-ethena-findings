## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Avoid contract existence checks by using low level calls | 1 |
| [GAS-2](#GAS-2) | Cache state variables to save gas | 25 |
| [GAS-3](#GAS-3) | State variables should be cached in stack variables | 3 |
| [GAS-4](#GAS-4) | Function Call Caching | 27 |
| [GAS-5](#GAS-5) | State variables only set in the constructor should be declared immutable | 1 |
| [GAS-6](#GAS-6) | Optimize gas costs using >= and <= operators | 16 |
| [GAS-7](#GAS-7) | Simple checks for zero can be done using assembly to save gas | 3 |
| [GAS-8](#GAS-8) | State variables should be cached in stack variables | 3 |
| [GAS-9](#GAS-9) | Upgrade to Solidity 0.8.19 or Later for Gas Optimization | 6 |
| [GAS-10](#GAS-10) | Use assembly to check zero | 7 |
| [GAS-11](#GAS-11) | Use assembly to emit events, in order to save gas | 19 |
| [GAS-12](#GAS-12) | Use assembly to write address/contract type storage values | 47 |
### <a name="GAS-1"></a>[GAS-1] Avoid contract existence checks by using low level calls

#### Impact:
Using low level calls can help avoid unnecessary contract existence checks, resulting in lower gas costs.

*Instances (1)*:
```solidity
File: Ethena Labs/EthenaMinting.sol

341:     address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);

```

### <a name="GAS-2"></a>[GAS-2] Cache state variables to save gas

#### Impact:
Caching state variables in stack variables can save gas by reducing read operations.

*Instances (25)*:
```solidity
File: Ethena Labs/EthenaMinting.sol

28:   bytes32 private constant EIP712_DOMAIN =

32:   bytes32 private constant ROUTE_TYPE = keccak256("Route(address[] addresses,uint256[] ratios)");

35:   bytes32 private constant ORDER_TYPE = keccak256(

40:   bytes32 private constant MINTER_ROLE = keccak256("MINTER_ROLE");

43:   bytes32 private constant REDEEMER_ROLE = keccak256("REDEEMER_ROLE");

46:   bytes32 private constant GATEKEEPER_ROLE = keccak256("GATEKEEPER_ROLE");

49:   bytes32 private constant EIP712_DOMAIN_TYPEHASH = keccak256(abi.encodePacked(EIP712_DOMAIN));

55:   bytes32 private constant EIP_712_NAME = keccak256("EthenaMinting");

58:   bytes32 private constant EIP712_REVISION = keccak256("1");

143:     _domainSeparator = _computeDomainSeparator();

340:     bytes32 taker_order_hash = hashOrder(order);

379:     uint256 invalidatorSlot = uint64(nonce) >> 8;

422:     IERC20 token = IERC20(asset);

```

```solidity
File: Ethena Labs/StakedUSDe.sol

26:   bytes32 private constant REWARDER_ROLE = keccak256("REWARDER_ROLE");

28:   bytes32 private constant BLACKLIST_MANAGER_ROLE = keccak256("BLACKLIST_MANAGER_ROLE");

30:   bytes32 private constant SOFT_RESTRICTED_STAKER_ROLE = keccak256("SOFT_RESTRICTED_STAKER_ROLE");

32:   bytes32 private constant FULL_RESTRICTED_STAKER_ROLE = keccak256("FULL_RESTRICTED_STAKER_ROLE");

150:       uint256 amountToDistribute = balanceOf(from);

192:     uint256 _totalSupply = totalSupply();

```

```solidity
File: Ethena Labs/StakedUSDeV2.sol

43:     silo = new USDeSilo(address(this), address(_asset));

98:     uint256 shares = previewWithdraw(assets);

100:     cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;

114:     uint256 assets = previewRedeem(shares);

116:     cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;

```

```solidity
File: Ethena Labs/USDeSilo.sol

22:         USDE = IERC20(usde);

```

### <a name="GAS-3"></a>[GAS-3] State variables should be cached in stack variables

#### Impact:
Re-reading state variables from storage during a function call can result in higher gas costs. Caching state variables in stack variables can be a more gas-efficient approach.

*Instances (3)*:
```solidity
File: Ethena Labs/EthenaMinting.sol

142:     _chainId = block.chainid;

```

```solidity
File: Ethena Labs/StakedUSDe.sol

94:     lastDistributionTimestamp = block.timestamp;

```

```solidity
File: Ethena Labs/StakedUSDeV2.sol

80:     uint256 assets = userCooldown.underlyingAmount;

```

### <a name="GAS-4"></a>[GAS-4] Function Call Caching

#### Impact:
Caching function results reduces gas consumption by avoiding repeated calls.

*Instances (27)*:
```solidity
File: Ethena Labs/EthenaMinting.sol

49:   bytes32 private constant EIP712_DOMAIN_TYPEHASH = keccak256(abi.encodePacked(EIP712_DOMAIN));

178:     usde.mint(order.beneficiary, order.usde_amount);

206:     usde.burnFrom(order.benefactor, order.usde_amount);

248:     if (wallet == address(0) || !_custodianAddresses.contains(wallet)) revert InvalidAddress();

260:     if (!_supportedAssets.remove(asset)) revert InvalidAssetAddress();

266:     return _supportedAssets.contains(asset);

271:     if (!_custodianAddresses.remove(custodian)) revert InvalidCustodianAddress();

291:     if (asset == address(0) || asset == address(usde) || !_supportedAssets.add(asset)) {

299:     if (custodian == address(0) || custodian == address(usde) || !_custodianAddresses.add(custodian)) {

317:     return ECDSA.toTypedDataHash(getDomainSeparator(), keccak256(encodeOrder(order)));

321:     return abi.encode(

335:     return abi.encode(ROUTE_TYPE, route.addresses, route.ratios);

341:     address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);

364:       if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)

407:       if (!_supportedAssets.contains(asset)) revert UnsupportedAsset();

421:     if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();

426:       token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);

431:       token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);

452:     return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));

```

```solidity
File: Ethena Labs/SingleAdminAccessControl.sol

59:     super.renounceRole(role, account);

79:     super._grantRole(role, account);

```

```solidity
File: Ethena Labs/StakedUSDe.sol

213:     super._deposit(caller, receiver, assets, shares);

236:     super._withdraw(caller, receiver, _owner, assets, shares);

```

```solidity
File: Ethena Labs/StakedUSDeV2.sol

59:     return super.withdraw(assets, receiver, owner);

72:     return super.redeem(shares, receiver, owner);

86:       silo.withdraw(receiver, assets);

```

```solidity
File: Ethena Labs/USDeSilo.sol

31:         USDE.transfer(to, amount);

```

### <a name="GAS-5"></a>[GAS-5] State variables only set in the constructor should be declared immutable

#### Impact:
Avoids a Gsset (20000 gas) in the constructor and reduces gas costs for state variable accesses.

*Instances (1)*:
```solidity
File: Ethena Labs/EthenaMinting.sol

111:   constructor(

```

### <a name="GAS-6"></a>[GAS-6] Optimize gas costs using >= and <= operators

#### Impact:
Using >= and <= operators can save gas compared to > and < operators.

*Instances (16)*:
```solidity
File: Ethena Labs/EthenaMinting.sol

98:     if (mintedPerBlock[block.number] + mintAmount > maxMintPerBlock) revert MaxMintPerBlockExceeded();

105:     if (redeemedPerBlock[block.number] + redeemAmount > maxRedeemPerBlock) revert MaxRedeemPerBlockExceeded();

126:     for (uint256 i = 0; i < _assets.length; i++) {

130:     for (uint256 j = 0; j < _custodians.length; j++) {

346:     if (block.timestamp > order.expiry) revert SignatureExpired();

363:     for (uint256 i = 0; i < route.addresses.length; ++i) {

403:       if (address(this).balance < amount) revert InvalidAmount();

424:     for (uint256 i = 0; i < addresses.length; ++i) {

430:     if (remainingBalance > 0) {

```

```solidity
File: Ethena Labs/StakedUSDe.sol

176:     if (timeSinceLastDistribution >= VESTING_PERIOD) {

193:     if (_totalSupply > 0 && _totalSupply < MIN_SHARES) revert MinSharesViolation();

193:     if (_totalSupply > 0 && _totalSupply < MIN_SHARES) revert MinSharesViolation();

```

```solidity
File: Ethena Labs/StakedUSDeV2.sol

82:     if (block.timestamp >= userCooldown.cooldownEnd) {

96:     if (assets > maxWithdraw(owner)) revert ExcessiveWithdrawAmount();

112:     if (shares > maxRedeem(owner)) revert ExcessiveRedeemAmount();

127:     if (duration > MAX_COOLDOWN_DURATION) {

```

### <a name="GAS-7"></a>[GAS-7] Simple checks for zero can be done using assembly to save gas

#### Impact:
Using assembly for simple zero checks can save gas compared to the equivalent high-level Solidity code.

*Instances (3)*:
```solidity
File: Ethena Labs/EthenaMinting.sol

378:     if (nonce == 0) revert InvalidNonce();

```

```solidity
File: Ethena Labs/StakedUSDe.sol

51:     if (amount == 0) revert InvalidAmount();

```

```solidity
File: Ethena Labs/StakedUSDeV2.sol

34:     if (cooldownDuration == 0) revert OperationNotAllowed();

```

### <a name="GAS-8"></a>[GAS-8] State variables should be cached in stack variables

#### Impact:
Re-reading state variables from storage during a function call can result in higher gas costs. Caching state variables in stack variables can be a more gas-efficient approach.

*Instances (3)*:
```solidity
File: Ethena Labs/EthenaMinting.sol

142:     _chainId = block.chainid;

```

```solidity
File: Ethena Labs/StakedUSDe.sol

94:     lastDistributionTimestamp = block.timestamp;

```

```solidity
File: Ethena Labs/StakedUSDeV2.sol

80:     uint256 assets = userCooldown.underlyingAmount;

```

### <a name="GAS-9"></a>[GAS-9] Upgrade to Solidity 0.8.19 or Later for Gas Optimization

#### Impact:
Upgrading to Solidity 0.8.19 or a later version may optimize gas usage.

*Instances (6)*:
```solidity
File: Ethena Labs/EthenaMinting.sol

2: pragma solidity 0.8.19;

```

```solidity
File: Ethena Labs/SingleAdminAccessControl.sol

2: pragma solidity 0.8.19;

```

```solidity
File: Ethena Labs/StakedUSDe.sol

2: pragma solidity 0.8.19;

```

```solidity
File: Ethena Labs/StakedUSDeV2.sol

2: pragma solidity 0.8.19;

```

```solidity
File: Ethena Labs/USDe.sol

2: pragma solidity 0.8.19;

```

```solidity
File: Ethena Labs/USDeSilo.sol

2: pragma solidity ^0.8.0;

```

### <a name="GAS-10"></a>[GAS-10] Use assembly to check zero

#### Impact:
Using assembly to check for zero can save gas by allowing more direct access to the EVM and reducing the overhead associated with high-level operations in Solidity.

*Instances (7)*:
```solidity
File: Ethena Labs/EthenaMinting.sol

120:     if (_assets.length == 0) revert NoAssetsProvided();

344:     if (order.collateral_amount == 0) revert InvalidAmount();

345:     if (order.usde_amount == 0) revert InvalidAmount();

360:     if (route.addresses.length == 0) {

378:     if (nonce == 0) revert InvalidNonce();

```

```solidity
File: Ethena Labs/StakedUSDe.sol

51:     if (amount == 0) revert InvalidAmount();

```

```solidity
File: Ethena Labs/StakedUSDeV2.sol

34:     if (cooldownDuration == 0) revert OperationNotAllowed();

```

### <a name="GAS-11"></a>[GAS-11] Use assembly to emit events, in order to save gas

#### Impact:
Using assembly to emit events can save gas by making use of scratch space and free memory pointers, avoiding memory expansion costs.

*Instances (19)*:
```solidity
File: Ethena Labs/EthenaMinting.sol

145:     emit USDeSet(address(_usde));

154:     emit Received(msg.sender, msg.value);

179:     emit Mint(

208:     emit Redeem(

237:     emit DelegatedSignerAdded(_delegateTo, msg.sender);

243:     emit DelegatedSignerRemoved(_removedSigner, msg.sender);

255:     emit CustodyTransfer(wallet, asset, amount);

261:     emit AssetRemoved(asset);

272:     emit CustodianAddressRemoved(custodian);

294:     emit AssetAdded(asset);

302:     emit CustodianAddressAdded(custodian);

439:     emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);

446:     emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);

```

```solidity
File: Ethena Labs/SingleAdminAccessControl.sol

28:     emit AdminTransferRequested(_currentDefaultAdmin, newAdmin);

74:       emit AdminTransferred(_currentDefaultAdmin, account);

```

```solidity
File: Ethena Labs/StakedUSDe.sol

98:     emit RewardsReceived(amount, newVestingAmount);

155:       emit LockedAmountRedistributed(from, to, amountToDistribute);

```

```solidity
File: Ethena Labs/StakedUSDeV2.sol

133:     emit CooldownDurationUpdated(previousDuration, cooldownDuration);

```

```solidity
File: Ethena Labs/USDe.sol

24:     emit MinterUpdated(newMinter, minter);

```

### <a name="GAS-12"></a>[GAS-12] Use assembly to write address/contract type storage values

#### Impact:
Using assembly to write address or contract type storage values can save gas.

*Instances (47)*:
```solidity
File: Ethena Labs/EthenaMinting.sol

40:   bytes32 private constant MINTER_ROLE = keccak256("MINTER_ROLE");

43:   bytes32 private constant REDEEMER_ROLE = keccak256("REDEEMER_ROLE");

46:   bytes32 private constant GATEKEEPER_ROLE = keccak256("GATEKEEPER_ROLE");

49:   bytes32 private constant EIP712_DOMAIN_TYPEHASH = keccak256(abi.encodePacked(EIP712_DOMAIN));

52:   address private constant NATIVE_TOKEN = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;

55:   bytes32 private constant EIP_712_NAME = keccak256("EthenaMinting");

58:   bytes32 private constant EIP712_REVISION = keccak256("1");

122:     usde = _usde;

126:     for (uint256 i = 0; i < _assets.length; i++) {

130:     for (uint256 j = 0; j < _custodians.length; j++) {

142:     _chainId = block.chainid;

143:     _domainSeparator = _computeDomainSeparator();

340:     bytes32 taker_order_hash = hashOrder(order);

356:     uint256 totalRatio = 0;

363:     for (uint256 i = 0; i < route.addresses.length; ++i) {

381:     mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];

382:     uint256 invalidator = invalidatorStorage[invalidatorSlot];

393:     mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];

422:     IERC20 token = IERC20(asset);

423:     uint256 totalTransferred = 0;

424:     for (uint256 i = 0; i < addresses.length; ++i) {

437:     uint256 oldMaxMintPerBlock = maxMintPerBlock;

438:     maxMintPerBlock = _maxMintPerBlock;

444:     uint256 oldMaxRedeemPerBlock = maxRedeemPerBlock;

445:     maxRedeemPerBlock = _maxRedeemPerBlock;

```

```solidity
File: Ethena Labs/SingleAdminAccessControl.sol

27:     _pendingDefaultAdmin = newAdmin;

76:       _currentDefaultAdmin = account;

```

```solidity
File: Ethena Labs/StakedUSDe.sol

26:   bytes32 private constant REWARDER_ROLE = keccak256("REWARDER_ROLE");

28:   bytes32 private constant BLACKLIST_MANAGER_ROLE = keccak256("BLACKLIST_MANAGER_ROLE");

30:   bytes32 private constant SOFT_RESTRICTED_STAKER_ROLE = keccak256("SOFT_RESTRICTED_STAKER_ROLE");

32:   bytes32 private constant FULL_RESTRICTED_STAKER_ROLE = keccak256("FULL_RESTRICTED_STAKER_ROLE");

93:     vestingAmount = newVestingAmount;

94:     lastDistributionTimestamp = block.timestamp;

150:       uint256 amountToDistribute = balanceOf(from);

192:     uint256 _totalSupply = totalSupply();

```

```solidity
File: Ethena Labs/StakedUSDeV2.sol

44:     cooldownDuration = MAX_COOLDOWN_DURATION;

79:     UserCooldown storage userCooldown = cooldowns[msg.sender];

80:     uint256 assets = userCooldown.underlyingAmount;

83:       userCooldown.cooldownEnd = 0;

84:       userCooldown.underlyingAmount = 0;

98:     uint256 shares = previewWithdraw(assets);

114:     uint256 assets = previewRedeem(shares);

131:     uint24 previousDuration = cooldownDuration;

132:     cooldownDuration = duration;

```

```solidity
File: Ethena Labs/USDe.sol

25:     minter = newMinter;

```

```solidity
File: Ethena Labs/USDeSilo.sol

21:         STAKING_VAULT = stakingVault;

22:         USDE = IERC20(usde);

```