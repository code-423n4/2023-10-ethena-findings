# Gas Optimization

# Summary

| Number | Gas Optimization                                                                                                                             | Context |
| :----: | :------------------------------------------------------------------------------------------------------------------------------------------- | :-----: |
| [G-01] | Unnecessary casting as variable is already of the same type                                                                                  |    5    |
| [G-02] | ++i/i++ should be unchecked{++i}/unchecked{i++} when it's not possible for them to overflow, as is the case when used in for and while-loops |    4    |
| [G-03] | Use hardcode address instead address(this)                                                                                                   |    5    |
| [G-04] | <X> += <Y> costs more gas than <X> = <X> + <Y> for state variables                                                                           |    4    |
| [G-05] | xpressions for constant values such as a call to keccak256(), should use immutable rather than constant                                      |   12    |
| [G-06] | A modifier used only once and not being inherited should be inlined to save gas                                                              |    4    |
| [G-07] | Using > 0 costs more gas than != 0                                                                                                           |    3    |
| [G-08] | abi.encode() is less efficient than abi.encodePacked()                                                                                       |    3    |
| [G-09] | State variables which are not modified within functions should be set as constants or immutable for values set at deployment                 |    2    |
| [G-10] | Use assembly for math (add, sub, mul, div)                                                                                                   |    9    |
| [G-11] | USE BITMAPS TO SAVE GAS                                                                                                                      |    3    |
| [G-12] | Avoid contract existence checks by using low level calls                                                                                     |    8    |
| [G-13] | Use Short Circuiting rules to your advantage                                                                                                 |   11    |
| [G-14] | admin functions no uses of the nonReentrant modifier                                                                                         |    3    |
| [G-15] | Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4                                                        |    1    |
| [G-16] | Should use arguments instead of state variable                                                                                               |    6    |
| [G-17] | The result of function calls should be cached rather than re-calling the function                                                            |    2    |
| [G-18] | Use assembly to perform efficient back-to-back calls                                                                                         |    9    |

## [G-01] Unnecessary casting as variable is already of the same type

```solidity
File: contracts/StakedUSDe.sol

139    if (address(token) == asset()) revert InvalidToken();

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L139C1-L140C1

StakedUSDeV2.sol.cooldownAssets(): silo should not be cast to address as it’s declared as an address

```solidity
File: contracts/StakedUSDeV2.sol

103    _withdraw(_msgSender(), address(silo), owner, assets, shares);

119    _withdraw(_msgSender(), address(silo), owner, assets, shares);
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L103

```solidity
File: contracts/EthenaMinting.sol


291    if (asset == address(0) || asset == address(usde) || !_supportedAssets.add(asset)) {

299    if (custodian == address(0) || custodian == address(usde) || !_custodianAddresses.add(custodian)) {
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L291C1

## [G-02] ++i/i++ should be unchecked{++i}/unchecked{i++} when it's not possible for them to overflow, as is the case when used in for and while-loops

When you're sure that an operation won't lead to overflow or underflow, you can use unchecked arithmetic to potentially save gas and increase efficiency. For example, in cases where ++i and i++ won't result in overflow or underflow, you can use unchecked{++i} and unchecked{i++} to indicate that these operations are safe.

```solidity
File: contracts/EthenaMinting.sol

126    for (uint256 i = 0; i < _assets.length; i++) {

130    for (uint256 j = 0; j < _custodians.length; j++) {

363    for (uint256 i = 0; i < route.addresses.length; ++i) {

424    for (uint256 i = 0; i < addresses.length; ++i) {
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L126

## [G-03] Use hardcode address instead address(this)

It can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

```solidity
File: contracts/StakedUSDe.sol

96    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

167    return IERC20(asset()).balanceOf(address(this)) - getUnvestedAmount();
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L167

```solidity
File: contracts/StakedUSDeV2.sol

43    silo = new USDeSilo(address(this), address(_asset));
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L43

```solidity
File: contracts/EthenaMinting.sol

403      if (address(this).balance < amount) revert InvalidAmount();

452    return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L403

## [G-04] <X> += <Y> costs more gas than <X> = <X> + <Y> for state variables

```solidity
File: contracts/EthenaMinting.sol

174    mintedPerBlock[block.number] += order.usde_amount;

205    redeemedPerBlock[block.number] += order.usde_amount;

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L174

```solidity
File: contracts/StakedUSDeV2.sol

101    cooldowns[owner].underlyingAmount += assets;

117    cooldowns[owner].underlyingAmount += assets;
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L101

## [G-05] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

The immutable keyword was introduced to optimize the storage of constant values, especially those that can be computed at compile-time, like the result of keccak256(). When you use immutable, the value is computed at compile-time and included directly in the contract's bytecode. It becomes a part of the contract's storage and is available without any runtime computation. This approach reduces gas when accessing the constant value because there's no need to compute it at runtime. So by using immutable, you save gas because you eliminate the need for runtime computation of constant values. The value is readily available in the contract's bytecode.

```solidity
File: contracts/EthenaMinting.sol

28  bytes32 private constant EIP712_DOMAIN =
29    keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");

32  bytes32 private constant ROUTE_TYPE = keccak256("Route(address[] addresses,uint256[] ratios)");


35  bytes32 private constant ORDER_TYPE = keccak256(
36    "Order(uint8 order_type,uint256 expiry,uint256 nonce,address benefactor,address beneficiary,address collateral_asset,uint256 collateral_amount,uint256 usde_amount)"
37  );


40  bytes32 private constant MINTER_ROLE = keccak256("MINTER_ROLE");

43  bytes32 private constant REDEEMER_ROLE = keccak256("REDEEMER_ROLE");

46  bytes32 private constant GATEKEEPER_ROLE = keccak256("GATEKEEPER_ROLE");

49  bytes32 private constant EIP712_DOMAIN_TYPEHASH = keccak256(abi.encodePacked(EIP712_DOMAIN));

55  bytes32 private constant EIP_712_NAME = keccak256("EthenaMinting");

58  bytes32 private constant EIP712_REVISION = keccak256("1");
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L28

```solidity
File: contracts/StakedUSDe.sol

26  bytes32 private constant REWARDER_ROLE = keccak256("REWARDER_ROLE");

30  bytes32 private constant BLACKLIST_MANAGER_ROLE = keccak256("BLACKLIST_MANAGER_ROLE");

32  bytes32 private constant SOFT_RESTRICTED_STAKER_ROLE = keccak256("SOFT_RESTRICTED_STAKER_ROLE");
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L26

## [G-06] A modifier used only once and not being inherited should be inlined to save gas

Inlining a modifier that is used only once and not inherited can help save gas by reducing the overhead of function call and stack management associated with applying a modifier. Inlining essentially means directly including the code of the modifier within the function where it's used. This is a manual optimization that can be applied in cases where the modifier is simple and only used in one place.

```solidity
File: contracts/EthenaMinting.sol

97  modifier belowMaxMintPerBlock(uint256 mintAmount) {

104  modifier belowMaxRedeemPerBlock(uint256 redeemAmount) {
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L97-L107

```solidity
File: contracts/USDeSilo.sol

23  modifier onlyStakingVault() {
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L23-L26

## [G-07] Using > 0 costs more gas than != 0

Using != 0 is more gas-efficient than > 0 when checking if a value is non-zero because the != comparison directly checks for inequality, resulting in lower gas costs.

```solidity
File: contracts/StakedUSDe.sol

90    if (getUnvestedAmount() > 0) revert StillVesting();

193    if (_totalSupply > 0 && _totalSupply < MIN_SHARES) revert MinSharesViolation();

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L90

```solidity
File: contracts/EthenaMinting.sol

430    if (remainingBalance > 0) {

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L430

## [G-08] abi.encode() is less efficient than abi.encodePacked()

abi.encode() is more gas-efficient when encoding function calls according to the Ethereum ABI. It includes the necessary function selectors and parameter encoding.abi.encodePacked() is more efficient for simply concatenating raw data without ABI-related encoding, making it a preferred choice when ABI structure isn't required.Your choice between the two functions should be based on whether you need ABI compliance for contract interactions or a more lightweight data concatenation.

```solidity
File: contracts/EthenaMinting.sol

321    return abi.encode(

335    return abi.encode(ROUTE_TYPE, route.addresses, route.ratios);

425    return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L321

## [G-09] State variables which are not modified within functions should be set as constants or immutable for values set at deployment

```solidity
File: contracts/EthenaMinting.sol

66  EnumerableSet.AddressSet internal _supportedAssets;

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L66

```solidity
File: contracts/StakedUSDeV2.sol

22  uint24 public MAX_COOLDOWN_DURATION = 90 days;

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L22

## [G-10] Use assembly for math (add, sub, mul, div)

Using assembly for math operations like addition, subtraction, multiplication, and division in Solidity can be more gas-efficient in certain cases.

```solidity
File: contracts/StakedUSDe.sol

91    uint256 newVestingAmount = amount + getUnvestedAmount();

167    return IERC20(asset()).balanceOf(address(this)) - getUnvestedAmount();

174    uint256 timeSinceLastDistribution = block.timestamp - lastDistributionTimestamp;

180    return ((VESTING_PERIOD - timeSinceLastDistribution) * vestingAmount) / VESTING_PERIOD;
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L91

```solidity
File: contracts/EthenaMinting.sol

98    if (mintedPerBlock[block.number] + mintAmount > maxMintPerBlock) revert MaxMintPerBlockExceeded();

105    if (redeemedPerBlock[block.number] + redeemAmount > maxRedeemPerBlock) revert MaxRedeemPerBlockExceeded();

425      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;

429    uint256 remainingBalance = amount - totalTransferred;

431      token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L98

## [G-11] USE BITMAPS TO SAVE GAS

```solidity
File: contracts/EthenaMinting.sol

86  mapping(address => mapping(address => bool)) public delegatedSigner;

236    delegatedSigner[_delegateTo][msg.sender] = true;

242    delegatedSigner[_removedSigner][msg.sender] = false;
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L86

## [G-12] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

```solidity
File: contracts/EthenaMinting.sol

178    usde.mint(order.beneficiary, order.usde_amount);

206    usde.burnFrom(order.benefactor, order.usde_amount);

253      IERC20(asset).safeTransfer(wallet, amount);

408      IERC20(asset).safeTransfer(beneficiary, amount);

426      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);

431      token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L178

```solidity
File: contracts/StakedUSDe.sol

96    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

140    IERC20(token).safeTransfer(to, amount);

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L96

## [G-13] Use Short Circuiting rules to your advantage

Short-Circuit Conditionals: If you have conditional statements, arrange them so that the most likely to be true come first. This can sometimes save gas by not needing to evaluate the rest of the conditions. This doesn't apply to your function as there are no conditional statements.

Use Short Circuiting rules to your advantage

```solidity
File: contracts/StakedUSDe.sol

75    if (_owner == address(0) || _initialRewarder == address(0) || address(_asset) == address(0)) {

149    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && !hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {

193    if (_totalSupply > 0 && _totalSupply < MIN_SHARES) revert MinSharesViolation();

246    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && to != address(0)) {

210    if (hasRole(SOFT_RESTRICTED_STAKER_ROLE, caller) || hasRole(SOFT_RESTRICTED_STAKER_ROLE, receiver)) {

232    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, caller) || hasRole(FULL_RESTRICTED_STAKER_ROLE, receiver)) {

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L75

```solidity
File:  contracts/EthenaMinting.sol

248    if (wallet == address(0) || !_custodianAddresses.contains(wallet)) revert InvalidAddress();

291    if (asset == address(0) || asset == address(usde) || !_supportedAssets.add(asset)) {

299    if (custodian == address(0) || custodian == address(usde) || !_custodianAddresses.add(custodian)) {

364      if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)

421    if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L248

## [G-14] admin functions no uses of the nonReentrant modifier

```solidity
File: contracts/EthenaMinting.sol

162  function mint(Order calldata order, Route calldata route, Signature calldata signature)
163    external
164    override
165    nonReentrant
166    onlyRole(MINTER_ROLE)
167    belowMaxMintPerBlock(order.usde_amount)
168  {



194  function redeem(Order calldata order, Signature calldata signature)
195    external
196    override
197    nonReentrant
198    onlyRole(REDEEMER_ROLE)
199    belowMaxRedeemPerBlock(order.usde_amount)
200  {


247  function transferToCustody(address wallet, address asset, uint256 amount) external nonReentrant onlyRole(MINTER_ROLE) {

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L162-L168

## [G-15] Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4

It's recommended to use the bytes.concat() function instead of abi.encodePacked() for concatenating byte arrays. bytes.concat() is a more gas-efficient and safer way to concatenate byte arrays, and it's considered a best practice in newer Solidity versions.

```solidity
File: contracts/EthenaMinting.sol

49  bytes32 private constant EIP712_DOMAIN_TYPEHASH = keccak256(abi.encodePacked(EIP712_DOMAIN));
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L49

## [G-16] Should use arguments instead of state variable

```solidity
File: contracts/StakedUSDeV2.sol

133    emit CooldownDurationUpdated(previousDuration, cooldownDuration);
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L133

```solidity
File: contracts/USDe.sol

24    emit MinterUpdated(newMinter, minter);
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L24

```solidity
File: contracts/SingleAdminAccessControl.sol

28    emit AdminTransferRequested(_currentDefaultAdmin, newAdmin);

74      emit AdminTransferred(_currentDefaultAdmin, account);
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol#L28

```solidity
File: contracts/EthenaMinting.sol

439    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);

446    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L439

## [G-17] The result of function calls should be cached rather than re-calling the function

Caching the result of function calls is a common gas optimization technique in Solidity. When a function's result doesn't change between multiple uses within the same transaction or block, caching can save gas by preventing redundant computations.

```solidity
File: contracts/StakedUSDe.sol

149    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && !hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {

210    if (hasRole(SOFT_RESTRICTED_STAKER_ROLE, caller) || hasRole(SOFT_RESTRICTED_STAKER_ROLE, receiver)) {
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L149

## [G-18] Use assembly to perform efficient back-to-back calls

```solidity
File:  contracts/EthenaMinting.sol

248    if (wallet == address(0) || !_custodianAddresses.contains(wallet)) revert InvalidAddress();

250      (bool success,) = wallet.call{value: amount}("");

253      IERC20(asset).safeTransfer(wallet, amount);




404      (bool success,) = (beneficiary).call{value: amount}("");

407      if (!_supportedAssets.contains(asset)) revert UnsupportedAsset();

408      IERC20(asset).safeTransfer(beneficiary, amount);




421    if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();

426      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);

431      token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);
```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L248