## Gas Optimizations

## Sumary

|   No | ISSUE  | INSTANCE  |
|------|--------|-----------|
|[G-01]|+= costs more gas than = + for state variable|6|
|[G-02]| Counting down in for statements is more gas efficient|4|
|[G-03]|Not using the named return variable when a function returns, wastes deployment gas|3|
|[G-04]|aching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)|10|
|[G-05]|Use assembly to perform efficient back-to-back calls|2|
|[G-06]|Using a positive conditional flow to save a NOT opcode|9|
|[G-07]| Don't initialize variables with default value|3|
|[G-08]|Modifiers are redundant if used only once or not used at all. |3|
|[G-09]|Use function instead of modifiers|5|
|[G-10]|Avoid emitting constants.|18|
|[G-11]|Unnecessary look up in if condition|9|
|[G-12]|abi.encode() is less efficient than abi.encodePacked()|3|
|[G-13]|Use hardcode address instead address(this)|5|


## [G-1]+= costs more gas than = + for state variables
use = + or = - instead to save gas

```solidity

file: contracts/EthenaMinting.sol

174     mintedPerBlock[block.number] += order.usde_amount;

205    redeemedPerBlock[block.number] += order.usde_amount;

368    totalRatio += route.ratios[i];

427   totalTransferred += amountToTransfer;
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L174
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L205
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L368
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L427


```solidity

file: contracts/StakedUSDeV2.sol

101     cooldowns[owner].underlyingAmount += assets;

117   cooldowns[owner].underlyingAmount += assets;
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L101
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L117

## [G-2] Counting down in for statements is more gas efficient
Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

```solidity

file: contracts/EthenaMinting.sol

126     for (uint256 i = 0; i < _assets.length; i++) 

130    for (uint256 j = 0; j < _custodians.length; j++)

363  for (uint256 i = 0; i < route.addresses.length; ++i) 

424  for (uint256 i = 0; i < addresses.length; ++i) 
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L126
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L130
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L363
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L424

## [G-3] Not using the named return variable when a function returns, wastes deployment gas

```solidity

file: contracts/EthenaMinting.sol

395  return valid;
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L395

```solidity


file: contracts/StakedUSDeV2.sol

105  return shares;

121  return assets;
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L105
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L121

## [G-4] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)

```solidity

file:main/contracts/USDe.sol

29   if (msg.sender != minter) revert OnlyMinter();
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L29

```solidity

file: contracts/EthenaMinting.sol

124   _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);

138  if (msg.sender != _admin) 

154   emit Received(msg.sender, msg.value);

180  msg.sender,

209     msg.sender,

236 delegatedSigner[_delegateTo][msg.sender] = true;
237     emit DelegatedSignerAdded(_delegateTo, msg.sender);

242  delegatedSigner[_removedSigner][msg.sender] = false;
243    emit DelegatedSignerRemoved(_removedSigner, msg.sender);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L124
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L138
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L154
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L180
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L209
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L236-L237 
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L242-L243

```solidity

file: contracts/StakedUSDe.sol

96   IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L96

## [G-5] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.

```solidity

file: contracts/USDe.sol

23  function setMinter(address newMinter) external onlyOwner {
    emit MinterUpdated(newMinter, minter);
    minter = newMinter;
  }

  function mint(address to, uint256 amount) external {
    if (msg.sender != minter) revert OnlyMinter();
    _mint(to, amount);
  }

  function renounceOwnership() public view override onlyOwner {
34    revert CantRenounceOwnership()
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L23-L34

```solidity

file: contracts/EthenaMinting.sol

219   function setMaxMintPerBlock(uint256 _maxMintPerBlock) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _setMaxMintPerBlock(_maxMintPerBlock);
  }

  /// @notice Sets the max redeemPerBlock limit
  function setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) external onlyRole(DEFAULT_ADMIN_ROLE) {
    _setMaxRedeemPerBlock(_maxRedeemPerBlock);
  }

  /// @notice Disables the mint and redeem
  function disableMintRedeem() external onlyRole(GATEKEEPER_ROLE) {
    _setMaxMintPerBlock(0);
    _setMaxRedeemPerBlock(0);
  }

  /// @notice Enables smart contracts to delegate an address for signing
  function setDelegatedSigner(address _delegateTo) external {
    delegatedSigner[_delegateTo][msg.sender] = true;
    emit DelegatedSignerAdded(_delegateTo, msg.sender);
  }

  /// @notice Enables smart contracts to undelegate an address for signing
  function removeDelegatedSigner(address _removedSigner) external {
    delegatedSigner[_removedSigner][msg.sender] = false;
    emit DelegatedSignerRemoved(_removedSigner, msg.sender);
  }

  /// @notice transfers an asset to a custody wallet
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

  /// @notice Removes an asset from the supported assets list
  function removeSupportedAsset(address asset) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (!_supportedAssets.remove(asset)) revert InvalidAssetAddress();
    emit AssetRemoved(asset);
  }

  /// @notice Checks if an asset is supported.
  function isSupportedAsset(address asset) external view returns (bool) {
    return _supportedAssets.contains(asset);
  }

  /// @notice Removes an custodian from the custodian address list
  function removeCustodianAddress(address custodian) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (!_custodianAddresses.remove(custodian)) revert InvalidCustodianAddress();
    emit CustodianAddressRemoved(custodian);
  }

  /// @notice Removes the minter role from an account, this can ONLY be executed by the gatekeeper role
  /// @param minter The address to remove the minter role from
  function removeMinterRole(address minter) external onlyRole(GATEKEEPER_ROLE) {
    _revokeRole(MINTER_ROLE, minter);
  }

  /// @notice Removes the redeemer role from an account, this can ONLY be executed by the gatekeeper role
  /// @param redeemer The address to remove the redeemer role from
  function removeRedeemerRole(address redeemer) external onlyRole(GATEKEEPER_ROLE) {
    _revokeRole(REDEEMER_ROLE, redeemer);
  }

  /* --------------- PUBLIC --------------- */

  /// @notice Adds an asset to the supported assets list.
  function addSupportedAsset(address asset) public onlyRole(DEFAULT_ADMIN_ROLE) {
    if (asset == address(0) || asset == address(usde) || !_supportedAssets.add(asset)) {
      revert InvalidAssetAddress();
    }
    emit AssetAdded(asset);
  }

  /// @notice Adds an custodian to the supported custodians list.
  function addCustodianAddress(address custodian) public onlyRole(DEFAULT_ADMIN_ROLE) {
    if (custodian == address(0) || custodian == address(usde) || !_custodianAddresses.add(custodian)) {
      revert InvalidCustodianAddress();
    }
    emit CustodianAddressAdded(custodian);
  }

  /// @notice Get the domain separator for the token
  /// @dev Return cached value if chainId matches cache, otherwise recomputes separator, to prevent replay attack across forks
  /// @return The domain separator of the token at current chain
  function getDomainSeparator() public view returns (bytes32) {
    if (block.chainid == _chainId) {
      return _domainSeparator;
    }
    return _computeDomainSeparator();
  }

  /// @notice hash an Order struct
  function hashOrder(Order calldata order) public view override returns (bytes32) {
    return ECDSA.toTypedDataHash(getDomainSeparator(), keccak256(encodeOrder(order)));
  }

  function encodeOrder(Order calldata order) public pure returns (bytes memory) {
    return abi.encode(
      ORDER_TYPE,
      order.order_type,
      order.expiry,
      order.nonce,
      order.benefactor,
      order.beneficiary,
      order.collateral_asset,
      order.collateral_amount,
      order.usde_amount
331    );

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L219-L331

# [G-6] Using a positive conditional flow to save a NOT opcode
Estimated savings: 3 gas

```solidity

file: contracts/EthenaMinting.sol

171  if (!verifyRoute(route, order.order_type)) revert InvalidRoute();
172    if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();

203  if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();

251  if (!success) revert TransferFailed();

260  if (!_supportedAssets.remove(asset)) revert InvalidAssetAddress();

271    if (!_custodianAddresses.remove(custodian)) revert InvalidCustodianAddress();

342  if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();

364   if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)

407   if (!_supportedAssets.contains(asset)) revert UnsupportedAsset();

421  if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L171-L172
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L203
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L251
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L260
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L271
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L342
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L364
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L407
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L421

### [G-7] Don't initialize variables with default value

```solidity

file: contracts/EthenaMinting.sol

356   uint256 totalRatio = 0;

423uint256 totalTransferred = 0;
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L356
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L423

```solidity

file: contracts/StakedUSDeV2.sol

83  userCooldown.cooldownEnd = 0;
84  userCooldown.underlyingAmount = 0;

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L83-84

### [G-8] Modifiers are redundant if used only once or not used at all. 
• Deployment. Gas Saved: 33 649
• Minumal Method Call. Gas Saved: -202
• Average Method Call. Gas Saved: 2 700
• Maximum Method Call. Gas Saved: 4 931

```solidity

file: contracts/EthenaMinting.sol

97   modifier belowMaxMintPerBlock(uint256 mintAmount) 

104  modifier belowMaxRedeemPerBlock(uint256 redeemAmount) 
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L97-L104

```solidity

file: contracts/StakedUSDe.sol

50  modifier notZero(uint256 amount) 

56  modifier notOwner(address target) 
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L50-L56

```solidity

file: contracts/StakedUSDeV2.sol

27  modifier ensureCooldownOff() 

33  modifier ensureCooldownOn() 
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L27-L33

## [G-9] Use function instead of modifiers 
Deployment. Gas Saved: 115 926
Minimum Method Call. Gas Saved: 162
Average Method Call. Gas Saved: -264
Maximum Method Call. Gas Saved: -481
Overall gas change: 734 (2.459%)

```solidity

file: contracts/EthenaMinting.sol

97   modifier belowMaxMintPerBlock(uint256 mintAmount) 

104  modifier belowMaxRedeemPerBlock(uint256 redeemAmount) 

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L97-L104

```solidity

file: contracts/StakedUSDe.sol

50 modifier notZero(uint256 amount) {
    if (amount == 0) revert InvalidAmount();
    _;
  }

  /// @notice ensures blacklist target is not owner
  modifier notOwner(address target)
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L50-L56

```solidity

file: 

27 modifier ensureCooldownOff() {
    if (cooldownDuration != 0) revert OperationNotAllowed();
    _;
  }

  /// @notice ensure cooldownDuration is gt 0
33  modifier ensureCooldownOn() 


```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L27-L33

```solidity

file: contracts/USDeSilo.sol

23  modifier onlyStakingVault()

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L23

```solidity

file: contracts/SingleAdminAccessControl.sol

17  modifier notAdmin(bytes32 role) 

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol#L17

## [G-10] Avoid emitting constants.
A log topic (declared with indexed) has a gas cost of Glogtopic (375 gas).

```solidity

file: contracts/USDe.sol

24  emit MinterUpdated(newMinter, minter);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L24

```solidity

file: contracts/EthenaMinting.so

145     emit USDeSet(address(_usde));
  }

  /* --------------- EXTERNAL --------------- */

  /**
   * @notice Fallback function to receive ether
   */
  receive() external payable {
154     emit Received(msg.sender, msg.value);

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L145-:154 

```solidity

file: contracts/EthenaMinting.sol#

179  emit Mint

208  emit Redeem(

237  emit DelegatedSignerAdded(_delegateTo, msg.sender);

243    emit DelegatedSignerRemoved(_removedSigner, msg.sender);

255  emit CustodyTransfer(wallet, asset, amount)

261  emit AssetRemoved(asset);

272  emit CustodianAddressRemoved(custodian);

294    emit AssetAdded(asset);

302 emit CustodianAddressAdded(custodian);

439    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);

446   emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock); 
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L179
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L208
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L237
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L243
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L255
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L261
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L272
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L294
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L302
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L439
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L446

```solidity

file: main/contracts/StakedUSDe.sol

98   emit RewardsReceived(amount, newVestingAmount);

155  emit LockedAmountRedistributed(from, to, amountToDistribute);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L98
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L155

```solidity

file: contracts/StakedUSDeV2.sol

133   emit CooldownDurationUpdated(previousDuration, cooldownDuration);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L133

```solidity

file: contracts/SingleAdminAccessControl.sol

28 

74   emit AdminTransferred(_currentDefaultAdmin, account);

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol#L28
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol#L74

## [G-11] Unnecessary look up in if condition

If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity

file: contracts/EthenaMinting.sol

248  if (wallet == address(0) || !_custodianAddresses.contains(wallet)) revert InvalidAddress();

291    if (asset == address(0) || asset == address(usde) || !_supportedAssets.add(asset)) 

299  if (custodian == address(0) || custodian == address(usde) || !_custodianAddresses.add(custodian)) 

342   if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();

364   if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)

421    if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L248
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L291
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L299
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L342
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L364
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L421

```solidity

file: contracts/StakedUSDe.sol

75 if (_owner == address(0) || _initialRewarder == address(0) || address(_asset) == address(0)) 

210 if (hasRole(SOFT_RESTRICTED_STAKER_ROLE, caller) || hasRole(SOFT_RESTRICTED_STAKER_ROLE, receiver))

232  if (hasRole(FULL_RESTRICTED_STAKER_ROLE, caller) || hasRole(FULL_RESTRICTED_STAKER_ROLE, receiver)) 
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L75
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L210
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L232

## [G-12] abi.encode() is less efficient than abi.encodePacked()

```solidity

file: contracts/EthenaMinting.sol

321 return abi.encode(
      ORDER_TYPE,
      order.order_type,
      order.expiry,
      order.nonce,
      order.benefactor,
      order.beneficiary,
      order.collateral_asset,
      order.collateral_amount,
      order.usde_amount
    );

335  return abi.encode(ROUTE_TYPE, route.addresses, route.ratios);

452   return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L321
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L335
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L452

## [G-13] Use hardcode address instead address(this)
Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

```solidity

file: main/contracts/EthenaMinting.sol

403   if (address(this).balance < amount) revert InvalidAmount();

452   return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L403
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L452

```solidity

file: contracts/StakedUSDe.sol

96   IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

167   return IERC20(asset()).balanceOf(address(this)) - getUnvestedAmount();
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L96
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L167

```solidity

file: contracts/StakedUSDeV2.sol

43  silo = new USDeSilo(address(this), address(_asset));

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L43