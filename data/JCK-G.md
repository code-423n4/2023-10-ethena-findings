
# Gas Optimizations

| Number | Issue | Instances | Total gas saved |
|--------|-------|-----------|-----------------|
|[G-01]| Possible Optimization EthenaMinting.sol  | 2 |    |
|[G-02]| Possible Optimization in EthenaMinting.sol  | 2 |    |
|[G-03]| Possible Optimization in verifyNonce() function   | 1 |    |
|[G-04]| Optimize the function redistributeLockedAmount()  | 1 |    |
|[G-05]| Optimize check order: avoid making any state reads/writes if we have to validate some function parameters  | 1 |    |
|[G-06]| public functions not called by the contract should be declared external instead  | 8 |    |
|[G-07]| Amounts should be checked for 0 before calling a transfer  | 3 |    |
|[G-08]| x += y costs more gas than x = x + y for state variables  | 6 | 18  |
|[G-09]| Do not initialize variables with default values  | 10 |  30  |
|[G-10]| Cache external calls outside of loop to avoid re-calling function on each iteration  | 1 |    |
|[G-11]| Use assembly to perform efficient back-to-back calls  | 2 |    |
|[G-12]| Use calldata instead of memory for function arguments that do not get mutated  | 1 |    |
|[G-13]| Use hardcoded address instead of address(this)  | 5 |    |
|[G-14]| Use uint256(1)/uint256(2) instead for true and false boolean states  | 20 | 200000  |
|[G-15]| Use assembly to validate msg.sender  | 5 |  15  |
|[G-16]| A modifier used only once and not being inherited should be inlined to save gas  | 2 |    |
|[G-17]| Use assembly for loops  | 3 |    |
|[G-18]| Should use arguments instead of state variable  | 3 |    |
|[G-19]| Can make the variable outside the loop to save gas  | 1 |  731  |
|[G-20]| abi.encode() is less efficient than abi.encodepacked()  | 3 | 153951   |
|[G-21]| Using delete statement can save gas  | 1 |    |
|[G-22]| Use Modifiers Instead of Functions To Save Gas  | 3 |    |
|[G-23]| Gas saving is achieved by removing the delete keyword (~60k)  | 1 |    |
|[G-24]| Counting down in for statements is more gas efficient  | 4 |  50868  |


## [G-01] Possible Optimization EthenaMinting.sol

In the mint function, there are multiple calls of  Order calldata order. This could be optimized by declaring it once at the beginning of the function and reusing the reference. This would save the gas used by the function call.

```solidity
file: contracts/EthenaMinting.sol

162 function mint(Order calldata order, Route calldata route, Signature calldata signature)
    external
    override
    nonReentrant
    onlyRole(MINTER_ROLE)
    belowMaxMintPerBlock(order.usde_amount)
  {
    if (order.order_type != OrderType.MINT) revert InvalidOrder();
    verifyOrder(order, signature);
    if (!verifyRoute(route, order.order_type)) revert InvalidRoute();
    if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();
    // Add to the minted amount in this block
    mintedPerBlock[block.number] += order.usde_amount;
    _transferCollateral(
      order.collateral_amount, order.collateral_asset, order.benefactor, route.addresses, route.ratios
    );
    usde.mint(order.beneficiary, order.usde_amount);
    emit Mint(
      msg.sender,
      order.benefactor,
      order.beneficiary,
      order.collateral_asset,
      order.collateral_amount,
      order.usde_amount
    );
  }

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L162-L187



[PASS] testDelegateSuccessfulMint() (gas: 241444)
[PASS] testDelegateSuccessfulMint() (gas: 240800)

### same issue in redeem() function

```solidity
file: contracts/EthenaMinting.sol

194   function redeem(Order calldata order, Signature calldata signature)
    external
    override
    nonReentrant
    onlyRole(REDEEMER_ROLE)
    belowMaxRedeemPerBlock(order.usde_amount)
  {
    if (order.order_type != OrderType.REDEEM) revert InvalidOrder();
    verifyOrder(order, signature);
    if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();
    // Add to the redeemed amount in this block
    redeemedPerBlock[block.number] += order.usde_amount;
    usde.burnFrom(order.benefactor, order.usde_amount);
    _transferToBeneficiary(order.beneficiary, order.collateral_asset, order.collateral_amount);
    emit Redeem(
      msg.sender,
      order.benefactor,
      order.beneficiary,
      order.collateral_asset,
      order.collateral_amount,
      order.usde_amount
    );
  }

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L194-216

[PASS] testDelegateSuccessfulRedeem() (gas: 325215)
[PASS] testDelegateSuccessfulRedeem() (gas: 324776)


### Same issue in verifyOrder() function

```solidity
file: contracts/EthenaMinting.sol

339     function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
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
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L339-L348


### recomondation

```solidity

    function mint(
        Order calldata order,
        Route calldata route,
        Signature calldata signature
    )
        external
        override
        nonReentrant
        onlyRole(MINTER_ROLE)
        belowMaxMintPerBlock(order.usde_amount)
    {
        Order memory my = order;
        if (my.order_type != OrderType.MINT) revert InvalidOrder();
        verifyOrder(order, signature);
        if (!verifyRoute(route, my.order_type)) revert InvalidRoute();
        if (!_deduplicateOrder(my.benefactor, my.nonce)) revert Duplicate();
        // Add to the minted amount in this block
        mintedPerBlock[block.number] += my.usde_amount;
        _transferCollateral(
            my.collateral_amount,
            my.collateral_asset,
            my.benefactor,
            route.addresses,
            route.ratios
        );
        usde.mint(my.beneficiary, my.usde_amount);
        emit Mint(
            msg.sender,
            my.benefactor,
            my.beneficiary,
            my.collateral_asset,
            my.collateral_amount,
            my.usde_amount
        );
    }
```

## [G-02] Possible Optimization in EthenaMinting.sol

### The verifyOrder functions check the order.collateral_amount and order.usde_amount with zero. instead of this checking you can checke order.collateral_amount and order.usde_amount with eache other can save gas because of removing of one if stamtent 

```solidity
file: main/contracts/EthenaMinting.sol

339   function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
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
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L339-L348

### Gas report 

[PASS] testCanUndelegate() (gas: 117886)
[PASS] testDelegateFailureMint() (gas: 111434)
[PASS] testDelegateFailureRedeem() (gas: 286158)
[PASS] testDelegateSuccessfulMint() (gas: 241444)
[PASS] testDelegateSuccessfulRedeem() (gas: 325215)

[PASS] testCanUndelegate() (gas: 117886)
[PASS] testDelegateFailureMint() (gas: 111434)
[PASS] testDelegateFailureRedeem() (gas: 286135)
[PASS] testDelegateSuccessfulMint() (gas: 241421)
[PASS] testDelegateSuccessfulRedeem() (gas: 325178)



### The constructor check the address(_usde) and _admin addresses. to address(0) if you ckeck address(_usde) and _admin with each other instead of ckecking them to address(0) you can remove one if statment and can save gas  

```solidity
file: contracts/EthenaMinting.sol

119    if (address(_usde) == address(0)) revert InvalidUSDeAddress();
       if (_assets.length == 0) revert NoAssetsProvided();
       if (_admin == address(0)) revert InvalidZeroAddress();
       usde = _usde;


```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L119-L122

## [G-03] Possible Optimization in verifyNonce() function 
 
### In the verifyNonce function, the contract performs a number of operations after checking if the invalidator & invalidatorBit in not  zero. If the invalidator & invalidatorBit is not  zero, these operations are unnecessary. By using a return statement after the revert, we can avoid these unnecessary operations.

```solidity
file: contracts/EthenaMinting.sol

377    function verifyNonce(address sender, uint256 nonce) public view override returns (bool, uint256, uint256, uint256) {
       if (nonce == 0) revert InvalidNonce();
       uint256 invalidatorSlot = uint64(nonce) >> 8;
       uint256 invalidatorBit = 1 << uint8(nonce);
       mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
       uint256 invalidator = invalidatorStorage[invalidatorSlot];
       if (invalidator & invalidatorBit != 0) revert InvalidNonce();
   
       return (true, invalidatorSlot, invalidator, invalidatorBit);
  }


```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L377-L386

[PASS] testCanUndelegate() (gas: 117886)
[PASS] testDelegateFailureMint() (gas: 111434)
[PASS] testDelegateFailureRedeem() (gas: 286158)
[PASS] testDelegateSuccessfulMint() (gas: 241444)
[PASS] testDelegateSuccessfulRedeem() (gas: 325215)


[PASS] testCanUndelegate() (gas: 117886)
[PASS] testDelegateFailureMint() (gas: 111434)
[PASS] testDelegateFailureRedeem() (gas: 98832)
[PASS] testDelegateSuccessfulMint() (gas: 136217)
[PASS] testDelegateSuccessfulRedeem() (gas: 98813)


## [G-04] Optimize the function redistributeLockedAmount()

### in function redistributeLockedAmount() berfor to execute _burn() first check the if (to != address(0)) _mint(to, amountToDistribute); condation because only the // to address of address(0) enables burning to save gas 

```solidity
file: contracts/StakedUSDe.sol

148    function redistributeLockedAmount(address from, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && !hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {
      uint256 amountToDistribute = balanceOf(from);
      _burn(from, amountToDistribute);
      // to address of address(0) enables burning
      if (to != address(0)) _mint(to, amountToDistribute);

      emit LockedAmountRedistributed(from, to, amountToDistribute);
    } else {
      revert OperationNotAllowed();
    }
  }

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L148-L159



berfore: [PASS] test_redistributeLockedAmount() (gas: 226215)
After:   [PASS] test_redistributeLockedAmount() (gas: 210297)


## [G-05]  Optimize check order: avoid making any state reads/writes if we have to validate some function parameters

```solidity
file: contracts/EthenaMinting.sol

111    constructor(
    IUSDe _usde,
    address[] memory _assets,
    address[] memory _custodians,
    address _admin,
    uint256 _maxMintPerBlock,
    uint256 _maxRedeemPerBlock
  ) {
    if (address(_usde) == address(0)) revert InvalidUSDeAddress();
    if (_assets.length == 0) revert NoAssetsProvided();
    if (_admin == address(0)) revert InvalidZeroAddress();
    usde = _usde;

    _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);

    for (uint256 i = 0; i < _assets.length; i++) {
      addSupportedAsset(_assets[i]);
    }

    for (uint256 j = 0; j < _custodians.length; j++) {
      addCustodianAddress(_custodians[j]);
    }

    // Set the max mint/redeem limits per block
    _setMaxMintPerBlock(_maxMintPerBlock);
    _setMaxRedeemPerBlock(_maxRedeemPerBlock);

    if (msg.sender != _admin) {
      _grantRole(DEFAULT_ADMIN_ROLE, _admin);
    }

    _chainId = block.chainid;
    _domainSeparator = _computeDomainSeparator();

    emit USDeSet(address(_usde));
  }

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L111-L146

We have a check for function parameters if msg.sender != _admin. If we look at our constructor, the operations that happen before this check consume so much gas, as they involve reading and writing states. first check this condation after chaking do other operations.


## [G-06] public functions not called by the contract should be declared external instead

when a function is declared as public, it is generated with an internal and an external interface. This means the function can be called both internally (within the contract) and externally (by other contracts or accounts). However, if a public function is never called internally and is only expected to be invoked externally, it is more gas-efficient to explicitly declare it as external.


```solidity
file: contracts/EthenaMinting.sol

334  function encodeRoute(Route calldata route) public pure returns (bytes memory) {


```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L334


```solidity
file: contracts/SingleAdminAccessControl.sol

41   function grantRole(bytes32 role, address account) public override onlyRole(DEFAULT_ADMIN_ROLE) notAdmin(role) 

50   function revokeRole(bytes32 role, address account) public override onlyRole(DEFAULT_ADMIN_ROLE) notAdmin(role) {

65   function owner() public view virtual returns (address) {

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol#L41


```solidity
file: contracts/StakedUSDe.sol

166   function totalAssets() public view override returns (uint256) {

184   function decimals() public pure override(ERC4626, ERC20) returns (uint8) {

257   function renounceRole(bytes32, address) public virtual override {

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L166


```solidity
file: contracts/USDe.sol

33    function renounceOwnership() public view override onlyOwner {

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L33

## [G-07] Amounts should be checked for 0 before calling a transfer

It can be beneficial to check if an amount is zero before invoking a transfer function, such as transfer or safeTransfer, to avoid unnecessary gas costs associated with executing the transfer operation. If the amount is zero, the transfer operation would have no effect, and performing the check can prevent unnecessary gas consumption.

```solidity
file:  contracts/EthenaMinting.sol   

253   IERC20(asset).safeTransfer(wallet, amount);

408   IERC20(asset).safeTransfer(beneficiary, amount);

426   token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L253


## [G-08] x += y costs more gas than x = x + y for state variables

```solidity
file: contracts/EthenaMinting.sol
 
174   mintedPerBlock[block.number] += order.usde_amount;

205   redeemedPerBlock[block.number] += order.usde_amount;

368   totalRatio += route.ratios[i];

427   totalTransferred += amountToTransfer;

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L174



```solidity
file: contracts/StakedUSDeV2.sol

101   cooldowns[owner].underlyingAmount += assets;

117   cooldowns[owner].underlyingAmount += assets;

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L101

## [G-09] Do not initialize variables with default values

```solidity
file: contracts/EthenaMinting.sol

236   delegatedSigner[_delegateTo][msg.sender] = true;

242   delegatedSigner[_removedSigner][msg.sender] = false;

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L236

```solidity
file: contracts/EthenaMinting.sol

126   for (uint256 i = 0; i < _assets.length; i++) {

130   for (uint256 j = 0; j < _custodians.length; j++) {

356   uint256 totalRatio = 0;

363   for (uint256 i = 0; i < route.addresses.length; ++i) {

423   uint256 totalTransferred = 0;

424   for (uint256 i = 0; i < addresses.length; ++i) {

```

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L126

```solidity
file:  contracts/StakedUSDeV2.sol

83    userCooldown.cooldownEnd = 0;'

84    userCooldown.underlyingAmount = 0;

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L83

## [G-10] Cache external calls outside of loop to avoid re-calling function on each iteration

Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.

```solidity
file: contracts/EthenaMinting.sol

424    for (uint256 i = 0; i < addresses.length; ++i) {
      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
      totalTransferred += amountToTransfer;
    }

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L424-L428

## [G-11] Use assembly to perform efficient back-to-back calls

If similar external calls are performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer), which can potentially allow us to avoid memory expansion costs. In this case, we are also able to efficiently store the function signatures together in memory as one word, saving multiple MLOADs in the process.

Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.


```solidity
file:  main/contracts/StakedUSDeV2.sol

100     cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
101     cooldowns[owner].underlyingAmount += assets;

116     cooldowns[owner].cooldownEnd = uint104(block.timestamp) + cooldownDuration;
117     cooldowns[owner].underlyingAmount += assets;

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L100-L101


## [G-12] Use calldata instead of memory for function arguments that do not get mutated

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

```solidity
file: contracts/EthenaMinting.sol

111     constructor(
    IUSDe _usde,
    address[] memory _assets,
    address[] memory _custodians,
    address _admin,
    uint256 _maxMintPerBlock,
    uint256 _maxRedeemPerBlock
  ) {

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L111-L118

## [G-13] Use hardcoded address instead of address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this. Refrences


```solidity
file:  contracts/EthenaMinting.sol

403    if (address(this).balance < amount) revert InvalidAmount();

452    return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L403


```solidity
file:  contracts/StakedUSDe.sol

96    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

167   return IERC20(asset()).balanceOf(address(this)) - getUnvestedAmount();

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L96


```solidity
file: contracts/StakedUSDeV2.sol

43   silo = new USDeSilo(address(this), address(_asset));

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L43

## [G-14] Use uint256(1)/uint256(2) instead for true and false boolean states

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. see source:
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27

```solidity
file: contracts/EthenaMinting.sol

86    mapping(address => mapping(address => bool)) public delegatedSigner;

250   (bool success,) = wallet.call{value: amount}("");

265   function isSupportedAsset(address asset) external view returns (bool) {

339   function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {

377   function verifyNonce(address sender, uint256 nonce) public view override returns (bool, uint256, uint256, uint256) {

391   function _deduplicateOrder(address sender, uint256 nonce) private returns (bool) {

392   (bool valid, uint256 invalidatorSlot, uint256 invalidator, uint256 invalidatorBit) = verifyNonce(sender, nonce);

404   (bool success,) = (beneficiary).call{value: amount}("");

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L86


```solidity
file: contracts/EthenaMinting.sol

236   delegatedSigner[_delegateTo][msg.sender] = true;

242   delegatedSigner[_removedSigner][msg.sender] = false;

347   return (true, taker_order_hash);

354   return true;

358   return false;

361   return false;

366   return false;

371   return false;

373   return true;

385   return (true, invalidatorSlot, invalidator, invalidatorBit);

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L236

```solidity
file:  contracts/StakedUSDe.sol

106    function addToBlacklist(address target, bool isFullBlacklisting)

120    function removeFromBlacklist(address target, bool isFullBlacklisting)

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L106

## [G-15] Use assembly to validate msg.sender

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

```solidity
file: contracts/EthenaMinting.sol

138   if (msg.sender != _admin) {

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L138


```solidity
file: contracts/SingleAdminAccessControl.sol

26    if (newAdmin == msg.sender) revert InvalidAdminChange();

32    if (msg.sender != _pendingDefaultAdmin) revert NotPendingAdmin();

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol#L26

```solidity
file: contracts/USDe.sol

29      if (msg.sender != minter) revert OnlyMinter();

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L29


```solidity
file: contracts/USDeSilo.sol

24   if (msg.sender != STAKING_VAULT) revert OnlyStakingVault();

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L24

## [G-16] A modifier used only once and not being inherited should be inlined to save gas

```solidity
file: contracts/EthenaMinting.sol

97    modifier belowMaxMintPerBlock(uint256 mintAmount) {

104   modifier belowMaxRedeemPerBlock(uint256 redeemAmount) {

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L97

## [G-17] Use assembly for loops

In the following instances, assembly is used for more gas efficient loops. The only memory slots that are manually used in the loops are scratch space (0x00-0x20), the free memory pointer (0x40), and the zero slot (0x60). This allows us to avoid using the free memory pointer to allocate new memory, which may result in memory expansion costs.

Note that in order to do this optimization safely we will need to cache and restore the free memory pointer after the loop. We will also set the zero slot (0x60) back to 0.

```solidity
file: contracts/EthenaMinting.sol

126     for (uint256 i = 0; i < _assets.length; i++) {
      addSupportedAsset(_assets[i]);
    }

130  for (uint256 j = 0; j < _custodians.length; j++) {
      addCustodianAddress(_custodians[j]);
    }

424   for (uint256 i = 0; i < addresses.length; ++i) {
      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
      totalTransferred += amountToTransfer;
    }

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L126-L128

## [G-18] Should use arguments instead of state variable

state variables should not used in emit , This will save near 97 gas

```solidity
file: contracts/EthenaMinting.sol

439   emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);

446   emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L439

```solidity
file: contracts/StakedUSDeV2.sol

133  emit CooldownDurationUpdated(previousDuration, cooldownDuration);

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L133

## [G-19] Can make the variable outside the loop to save gas

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here’s an example:

```solidity
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}

```

### Making variable inside the loop gas value:   662604
### Making variable outside the loop gas value:  661873

### Tools used Remix 


Consider making the stack variables before the loop which gonna save gas

```solidity
file: contracts/EthenaMinting.sol

424   for (uint256 i = 0; i < addresses.length; ++i) {
      uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
      token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
      totalTransferred += amountToTransfer;
    }

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L424-L428

## [G-20] abi.encode() is less efficient than abi.encodepacked()

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

```solidity
file: contracts/EthenaMinting.sol

321  return abi.encode(

335  return abi.encode(ROUTE_TYPE, route.addresses, route.ratios);

452  return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L321

## [G-21] Using delete statement can save gas

```solidity
file: contracts/EthenaMinting.sol

242  delegatedSigner[_removedSigner][msg.sender] = false;

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L236


## [G-22] Use Modifiers Instead of Functions To Save Gas

Example of two contracts with modifiers and internal view function:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Inlined {
    function isNotExpired(bool _true) internal view {
        require(_true == true, "Exchange: EXPIRED");
    }
function foo(bool _test) public returns(uint){
            isNotExpired(_test);
            return 1;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Modifier {
modifier isNotExpired(bool _true) {
        require(_true == true, "Exchange: EXPIRED");
        _;
    }
function foo(bool _test) public isNotExpired(_test)returns(uint){
        return 1;
    }
}

```

```solidity
file: main/contracts/SingleAdminAccessControl.sol

65    function owner() public view virtual returns (address) {
        return _currentDefaultAdmin;
    }


```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol#L65-L67


```solidity
file: main/contracts/SingleAdminAccessControl.sol

184  function decimals() public pure override(ERC4626, ERC20) returns (uint8) {
    return 18;
  }

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol#L184-L186


```solidity
file: main/contracts/EthenaMinting.sol
 
265  function isSupportedAsset(address asset) external view returns (bool) {
    return _supportedAssets.contains(asset);
  }

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L265-L267

## [G-23] Gas saving is achieved by removing the delete keyword (~60k)

30k gas savings were made by removing the delete keyword. The reason for using the delete keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the delete keyword is unnecessary. If it is removed, around 30k gas savings will be achieved.

```solidity
file: main/contracts/SingleAdminAccessControl.sol

92    delete _pendingDefaultAdmin;

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol#L92

## [G‑24] Counting down in for statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

by changing this logic you can save 12171 gas per one for loop 

Tools used Remix

```solidity
file: main/contracts/EthenaMinting.sol

126   for (uint256 i = 0; i < _assets.length; i++) {
   
139   for (uint256 j = 0; j < _custodians.length; j++) {

363   for (uint256 i = 0; i < route.addresses.length; ++i) {

424   for (uint256 i = 0; i < addresses.length; ++i) {

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L126