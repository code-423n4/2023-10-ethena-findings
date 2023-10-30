## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Contract should expose an interface | 58 |
| [NC-2](#NC-2) | State variables should include comments | 20 |
### <a name="NC-1"></a>[NC-1] Contract should expose an interface
All external/public functions should extend an interface to ensure the whole API is extracted.

*Instances (58)*:
```solidity
File: Ethena Labs/EthenaMinting.sol

162:   function mint(Order calldata order, Route calldata route, Signature calldata signature)

194:   function redeem(Order calldata order, Signature calldata signature)

219:   function setMaxMintPerBlock(uint256 _maxMintPerBlock) external onlyRole(DEFAULT_ADMIN_ROLE) {

224:   function setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) external onlyRole(DEFAULT_ADMIN_ROLE) {

229:   function disableMintRedeem() external onlyRole(GATEKEEPER_ROLE) {

235:   function setDelegatedSigner(address _delegateTo) external {

241:   function removeDelegatedSigner(address _removedSigner) external {

247:   function transferToCustody(address wallet, address asset, uint256 amount) external nonReentrant onlyRole(MINTER_ROLE) {

259:   function removeSupportedAsset(address asset) external onlyRole(DEFAULT_ADMIN_ROLE) {

265:   function isSupportedAsset(address asset) external view returns (bool) {

270:   function removeCustodianAddress(address custodian) external onlyRole(DEFAULT_ADMIN_ROLE) {

277:   function removeMinterRole(address minter) external onlyRole(GATEKEEPER_ROLE) {

283:   function removeRedeemerRole(address redeemer) external onlyRole(GATEKEEPER_ROLE) {

290:   function addSupportedAsset(address asset) public onlyRole(DEFAULT_ADMIN_ROLE) {

298:   function addCustodianAddress(address custodian) public onlyRole(DEFAULT_ADMIN_ROLE) {

308:   function getDomainSeparator() public view returns (bytes32) {

316:   function hashOrder(Order calldata order) public view override returns (bytes32) {

320:   function encodeOrder(Order calldata order) public pure returns (bytes memory) {

334:   function encodeRoute(Route calldata route) public pure returns (bytes memory) {

339:   function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {

351:   function verifyRoute(Route calldata route, OrderType orderType) public view override returns (bool) {

377:   function verifyNonce(address sender, uint256 nonce) public view override returns (bool, uint256, uint256, uint256) {

391:   function _deduplicateOrder(address sender, uint256 nonce) private returns (bool) {

401:   function _transferToBeneficiary(address beneficiary, address asset, uint256 amount) internal {

413:   function _transferCollateral(

436:   function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {

443:   function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {

451:   function _computeDomainSeparator() internal view returns (bytes32) {

```

```solidity
File: Ethena Labs/SingleAdminAccessControl.sol

25:   function transferAdmin(address newAdmin) external onlyRole(DEFAULT_ADMIN_ROLE) {

31:   function acceptAdmin() external {

41:   function grantRole(bytes32 role, address account) public override onlyRole(DEFAULT_ADMIN_ROLE) notAdmin(role) {

50:   function revokeRole(bytes32 role, address account) public override onlyRole(DEFAULT_ADMIN_ROLE) notAdmin(role) {

58:   function renounceRole(bytes32 role, address account) public virtual override notAdmin(role) {

65:   function owner() public view virtual returns (address) {

72:   function _grantRole(bytes32 role, address account) internal override {

```

```solidity
File: Ethena Labs/StakedUSDe.sol

89:   function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {

106:   function addToBlacklist(address target, bool isFullBlacklisting)

120:   function removeFromBlacklist(address target, bool isFullBlacklisting)

138:   function rescueTokens(address token, uint256 amount, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {

148:   function redistributeLockedAmount(address from, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {

166:   function totalAssets() public view override returns (uint256) {

173:   function getUnvestedAmount() public view returns (uint256) {

184:   function decimals() public pure override(ERC4626, ERC20) returns (uint8) {

191:   function _checkMinShares() internal view {

203:   function _deposit(address caller, address receiver, uint256 assets, uint256 shares)

225:   function _withdraw(address caller, address receiver, address _owner, uint256 assets, uint256 shares)

245:   function _beforeTokenTransfer(address from, address to, uint256) internal virtual override {

257:   function renounceRole(bytes32, address) public virtual override {

```

```solidity
File: Ethena Labs/StakedUSDeV2.sol

52:   function withdraw(uint256 assets, address receiver, address owner)

65:   function redeem(uint256 shares, address receiver, address owner)

78:   function unstake(address receiver) external {

95:   function cooldownAssets(uint256 assets, address owner) external ensureCooldownOn returns (uint256) {

111:   function cooldownShares(uint256 shares, address owner) external ensureCooldownOn returns (uint256) {

126:   function setCooldownDuration(uint24 duration) external onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: Ethena Labs/USDe.sol

23:   function setMinter(address newMinter) external onlyOwner {

28:   function mint(address to, uint256 amount) external {

33:   function renounceOwnership() public view override onlyOwner {

```

```solidity
File: Ethena Labs/USDeSilo.sol

30:     function withdraw(address to, uint256 amount) external onlyStakingVault {

```

### <a name="NC-2"></a>[NC-2] State variables should include comments
Consider adding comments to explain what critical state variables are supposed to do.

*Instances (20)*:
```solidity
File: Ethena Labs/EthenaMinting.sol

126:     for (uint256 i = 0; i < _assets.length; i++) {

130:     for (uint256 j = 0; j < _custodians.length; j++) {

356:     uint256 totalRatio = 0;

363:     for (uint256 i = 0; i < route.addresses.length; ++i) {

379:     uint256 invalidatorSlot = uint64(nonce) >> 8;

380:     uint256 invalidatorBit = 1 << uint8(nonce);

382:     uint256 invalidator = invalidatorStorage[invalidatorSlot];

423:     uint256 totalTransferred = 0;

424:     for (uint256 i = 0; i < addresses.length; ++i) {

425:       uint256 amountToTransfer = (amount * ratios[i]) / 10_000;

429:     uint256 remainingBalance = amount - totalTransferred;

437:     uint256 oldMaxMintPerBlock = maxMintPerBlock;

444:     uint256 oldMaxRedeemPerBlock = maxRedeemPerBlock;

```

```solidity
File: Ethena Labs/StakedUSDe.sol

91:     uint256 newVestingAmount = amount + getUnvestedAmount();

150:       uint256 amountToDistribute = balanceOf(from);

174:     uint256 timeSinceLastDistribution = block.timestamp - lastDistributionTimestamp;

192:     uint256 _totalSupply = totalSupply();

```

```solidity
File: Ethena Labs/StakedUSDeV2.sol

80:     uint256 assets = userCooldown.underlyingAmount;

98:     uint256 shares = previewWithdraw(assets);

114:     uint256 assets = previewRedeem(shares);

```


## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Constructor / Initialization Function Lacks Parameter Validation | 3 |
| [L-2](#L-2) | Avoid Usage of decimals() in ERC-20 Contracts | 1 |
| [L-3](#L-3) | Undetected Uninitialized Variables in Constructors | 3 |
| [L-4](#L-4) | Timestamp may be manipulation | 6 |
| [L-5](#L-5) | Consider implementing two-step procedure for updating protocol addresses | 37 |
### <a name="L-1"></a>[L-1] Constructor / Initialization Function Lacks Parameter Validation

#### Impact:
Constructors and initialization functions are critical in contracts, as they set initial states during deployment. Parameters passed to these functions directly affect the behavior of the contract/protocol. Failing to validate parameters can lead to system failures, abnormal behavior, instability, or security vulnerabilities. It's essential to thoroughly validate all parameters in constructors and initialization functions, and transactions should be rolled back if exceptions are found.

*Instances (3)*:
```solidity
File: Ethena Labs/StakedUSDeV2.sol

42:   constructor(IERC20 _asset, address initialRewarder, address owner) StakedUSDe(_asset, initialRewarder, owner) {

```

```solidity
File: Ethena Labs/USDe.sol

18:   constructor(address admin) ERC20("USDe", "USDe") ERC20Permit("USDe") {

```

```solidity
File: Ethena Labs/USDeSilo.sol

20:     constructor(address stakingVault, address usde) {

```

### <a name="L-2"></a>[L-2] Avoid Usage of decimals() in ERC-20 Contracts

#### Impact:
The decimals() function is not part of the ERC-20 standard and is an optional extension. Relying on it may lead to compatibility issues with tokens that do not support this function.

*Instances (1)*:
```solidity
File: Ethena Labs/StakedUSDe.sol

184:   function decimals() public pure override(ERC4626, ERC20) returns (uint8) {

```

### <a name="L-3"></a>[L-3] Undetected Uninitialized Variables in Constructors

#### Impact:
This issue identifies undetected uninitialized variables in constructors, which can lead to runtime errors or unexpected behavior. Ensure that all variables are properly initialized in the constructor.

*Instances (3)*:
```solidity
File: Ethena Labs/EthenaMinting.sol

122:     usde = _usde;

438:     maxMintPerBlock = _maxMintPerBlock;

445:     maxRedeemPerBlock = _maxRedeemPerBlock;

```

### <a name="L-4"></a>[L-4] Timestamp may be manipulation

#### Impact:
The block.timestamp can be manipulated by miners to perform MEV profiting or other time-based attacks.

*Instances (6)*:
```solidity
File: Ethena Labs/EthenaMinting.sol

346:     if (block.timestamp > order.expiry) revert SignatureExpired();

```

```solidity
File: Ethena Labs/StakedUSDe.sol

94:     lastDistributionTimestamp = block.timestamp;

174:     uint256 timeSinceLastDistribution = block.timestamp - lastDistributionTimestamp;

```

```solidity
File: Ethena Labs/StakedUSDeV2.sol

82:     if (block.timestamp >= userCooldown.cooldownEnd) {

100:     cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;

116:     cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;

```

### <a name="L-5"></a>[L-5] Consider implementing two-step procedure for updating protocol addresses

#### Impact:
A copy-paste error or a typo may end up bricking protocol functionality, or sending tokens to an address with no known private key. Consider implementing a two-step procedure for updating protocol addresses, where the recipient is set as pending and must 'accept' the assignment by making an affirmative call.

*Instances (37)*:
```solidity
File: Ethena Labs/EthenaMinting.sol

120:     if (_assets.length == 0) revert NoAssetsProvided();

135:     _setMaxMintPerBlock(_maxMintPerBlock);

136:     _setMaxRedeemPerBlock(_maxRedeemPerBlock);

219:   function setMaxMintPerBlock(uint256 _maxMintPerBlock) external onlyRole(DEFAULT_ADMIN_ROLE) {

220:     _setMaxMintPerBlock(_maxMintPerBlock);

224:   function setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) external onlyRole(DEFAULT_ADMIN_ROLE) {

225:     _setMaxRedeemPerBlock(_maxRedeemPerBlock);

230:     _setMaxMintPerBlock(0);

231:     _setMaxRedeemPerBlock(0);

235:   function setDelegatedSigner(address _delegateTo) external {

247:   function transferToCustody(address wallet, address asset, uint256 amount) external nonReentrant onlyRole(MINTER_ROLE) {

253:       IERC20(asset).safeTransfer(wallet, amount);

259:   function removeSupportedAsset(address asset) external onlyRole(DEFAULT_ADMIN_ROLE) {

260:     if (!_supportedAssets.remove(asset)) revert InvalidAssetAddress();

261:     emit AssetRemoved(asset);

265:   function isSupportedAsset(address asset) external view returns (bool) {

266:     return _supportedAssets.contains(asset);

290:   function addSupportedAsset(address asset) public onlyRole(DEFAULT_ADMIN_ROLE) {

291:     if (asset == address(0) || asset == address(usde) || !_supportedAssets.add(asset)) {

292:       revert InvalidAssetAddress();

294:     emit AssetAdded(asset);

407:       if (!_supportedAssets.contains(asset)) revert UnsupportedAsset();

408:       IERC20(asset).safeTransfer(beneficiary, amount);

421:     if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();

436:   function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {

443:   function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {

```

```solidity
File: Ethena Labs/StakedUSDe.sol

75:     if (_owner == address(0) || _initialRewarder == address(0) || address(_asset) == address(0)) {

96:     IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

139:     if (address(token) == asset()) revert InvalidToken();

166:   function totalAssets() public view override returns (uint256) {

167:     return IERC20(asset()).balanceOf(address(this)) - getUnvestedAmount();

```

```solidity
File: Ethena Labs/StakedUSDeV2.sol

42:   constructor(IERC20 _asset, address initialRewarder, address owner) StakedUSDe(_asset, initialRewarder, owner) {

95:   function cooldownAssets(uint256 assets, address owner) external ensureCooldownOn returns (uint256) {

96:     if (assets > maxWithdraw(owner)) revert ExcessiveWithdrawAmount();

114:     uint256 assets = previewRedeem(shares);

126:   function setCooldownDuration(uint24 duration) external onlyRole(DEFAULT_ADMIN_ROLE) {

```

```solidity
File: Ethena Labs/USDe.sol

23:   function setMinter(address newMinter) external onlyOwner {

```
