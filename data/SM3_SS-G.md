

# Gas Optimizations

| Number | Issue | Instances |
|--------|-------|-----------|
|[G-01]| Cache external calls outside of loop to avoid re-calling function on each iteration | 1 |
|[G-02]| Use assembly to perform efficient back-to-back calls | 2 |
|[G-03]| Use calldata instead of memory for function arguments that do not get mutated | 1 |
|[G-04]| Use hardcoded address instead of address(this) | 5 |
|[G-05]| Use assembly to validate msg.sender | 5 |
|[G-06]| A modifier used only once and not being inherited should be inlined to save gas | 2 |
|[G-07]| Use assembly for loops | 3 |
|[G-08]| Should use arguments instead of state variable | 3 |
|[G-09]| Can make the variable outside the loop to save gas |  |
|[G-10]| abi.encode() is less efficient than abi.encodepacked() | 3 |
|[G-11]| Using delete statement can save gas | 2 |
|[G-12]| Do not initialize variables with default values | 10 |


## [G-01] Cache external calls outside of loop to avoid re-calling function on each iteration

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

## [G-02] Use assembly to perform efficient back-to-back calls

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


## [G-03] Use calldata instead of memory for function arguments that do not get mutated

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

## [G-04] Use hardcoded address instead of address(this)

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



## [G-05] Use assembly to validate msg.sender

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

## [G-06] A modifier used only once and not being inherited should be inlined to save gas

```solidity
file: contracts/EthenaMinting.sol

97    modifier belowMaxMintPerBlock(uint256 mintAmount) {

104   modifier belowMaxRedeemPerBlock(uint256 redeemAmount) {

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L97

## [G-07] Use assembly for loops

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

## [G-08] Should use arguments instead of state variable

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

## [G-09] Can make the variable outside the loop to save gas

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

## [G-10] abi.encode() is less efficient than abi.encodepacked()

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

```solidity
file: contracts/EthenaMinting.sol

321  return abi.encode(

335  return abi.encode(ROUTE_TYPE, route.addresses, route.ratios);

452  return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L321

## [G-11] Using delete statement can save gas

```solidity
file: contracts/EthenaMinting.sol

236  delegatedSigner[_delegateTo][msg.sender] = true;

242  delegatedSigner[_removedSigner][msg.sender] = false;

```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L236

## [G-12] Do not initialize variables with default values

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