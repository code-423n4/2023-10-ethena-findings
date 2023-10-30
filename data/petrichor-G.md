# GAS OPTIMIZATION

|      |  issue  |  instance  |
|------|---------|------------|
|[G-01]|Amounts should be checked for 0 before calling a transfer|6|
|[G-02]|Unnecessary computation|2|
|[G-03]|keccak256() should only need to be called on a specific string literal once|3|
|[G-04]|abi.encode() is less efficient than abi.encodePacked()|1|
|[G-05]|With assembly, .call (bool success) transfer can be done gas-optimized|2|
|[G-06]| Use Assembly To Check For address(0)|3|
|[G-07]| A modifier used only once and not being inherited should be inlined to save gas|2|
|[G-08]|Expressions for constant values such as a call to keccak256(), should use immutable rather than constant|14|
|[G-09]|The result of function calls should be cached rather than re-calling the function|1|
|[G-10]|Can Make The Variable Outside The Loop To Save Gas |1|
|[G-11]|Duplicated require()/if() checks should be refactored to a modifier or function|1|
|[G-12]|Add unchecked {} for subtractions where the operands cannot underflow|1|









## [G-01] Amounts should be checked for 0 before calling a transfer

In Solidity, it is generally considered good practice to check for zero amounts before performing certain operations, such as transferring tokens or Ether, to save gas and prevent potential errors. This is particularly relevant when dealing with smart contracts that involve financial transactions.


```solidity
File:  contracts/EthenaMinting.sol
253   IERC20(asset).safeTransfer(wallet, amount);

408   IERC20(asset).safeTransfer(beneficiary, amount);

426   token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L253
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L408
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L426



```solidity
96    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

140   IERC20(token).safeTransfer(to, amount);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L96
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L140



```solidity
29    USDE.transfer(to, amount);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L29







## [G-02] Unnecessary computation

To save gas in Solidity, it is important to minimize unnecessary computations and optimize the code.

```solidity
    uint256 oldMaxMintPerBlock = maxMintPerBlock;
    maxMintPerBlock = _maxMintPerBlock;
    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L433-L439


```solidity
    uint256 oldMaxRedeemPerBlock = maxRedeemPerBlock;
    maxRedeemPerBlock = _maxRedeemPerBlock;
    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L444-L446

## [G-03] keccak256() should only need to be called on a specific string literal once

In Solidity, the keccak256 function is a hash function that takes an input and returns a 256-bit hash value. When using keccak256 on a specific string literal, it only needs to be called once and the result can be stored for later use to save gas.


 3 instances of this:
```solidity
  bytes32 private constant EIP712_DOMAIN =
    keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");

  /// @notice route type
  bytes32 private constant ROUTE_TYPE = keccak256("Route(address[] addresses,uint256[] ratios)");

  /// @notice order type
  bytes32 private constant ORDER_TYPE = keccak256(
    "Order(uint8 order_type,uint256 expiry,uint256 nonce,address benefactor,address beneficiary,address collateral_asset,uint256 collateral_amount,uint256 usde_amount)"
  );
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L28-L37





## [G-04] abi.encode() is less efficient than abi.encodePacked()

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. 

```solidity
File:  contracts/EthenaMinting.sol
452   return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L452


## [G-05] With assembly, .call (bool success) transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

```solidity
250     (bool success,) = wallet.call{value: amount}("");

404     (bool success,) = (beneficiary).call{value: amount}("");
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L250
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L404



## [G-06] Use Assembly To Check For address(0)


Using assembly in Solidity allows for low-level manipulation of the Ethereum Virtual Machine (EVM) and can be helpful in optimizing gas usage. You can leverage assembly to check for the address(0) or zero address, also known as the "null address," which typically represents an uninitialized or non-existent address. By checking for address(0) using assembly, you can save gas compared to using higher-level Solidity constructs.

```solidity
121  if (_admin == address(0)) revert InvalidZeroAddress();

343  if (order.beneficiary == address(0)) revert InvalidAmount();
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L121
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L343



```solidity
File:  contracts/USDe.sol
19   if (admin == address(0)) revert ZeroAddressException();
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L19



## [G-07] A modifier used only once and not being inherited should be inlined to save gas

By inlining a modifier that is used only once and not being inherited, you can eliminate the overhead of the generated code and reduce the gas cost of your contract.

```solidity
File:  contracts/EthenaMinting.sol
97   modifier belowMaxMintPerBlock(uint256 mintAmount) {

104  modifier belowMaxRedeemPerBlock(uint256 redeemAmount) {    
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L97
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L104



## [G-08] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

In Solidity, using the immutable keyword for expressions involving constant values, such as a call to keccak256(), can result in gas savings. The immutable keyword is used to declare state variables that are known at compile-time and cannot be modified after deployment. By using immutable for constant expressions, the compiler can optimize the deployment process and reduce gas costs.

There are 14 instances of this:
```solidity
File:  contracts/EthenaMinting.sol
  bytes32 private constant EIP712_DOMAIN =
    keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)");

  /// @notice route type
  bytes32 private constant ROUTE_TYPE = keccak256("Route(address[] addresses,uint256[] ratios)");

  /// @notice order type
  bytes32 private constant ORDER_TYPE = keccak256(
    "Order(uint8 order_type,uint256 expiry,uint256 nonce,address benefactor,address beneficiary,address collateral_asset,uint256 collateral_amount,uint256 usde_amount)"
  );

  /// @notice role enabling to invoke mint
  bytes32 private constant MINTER_ROLE = keccak256("MINTER_ROLE");

  /// @notice role enabling to invoke redeem
  bytes32 private constant REDEEMER_ROLE = keccak256("REDEEMER_ROLE");

  /// @notice role enabling to disable mint and redeem and remove minters and redeemers in an emergency
  bytes32 private constant GATEKEEPER_ROLE = keccak256("GATEKEEPER_ROLE");

  /// @notice EIP712 domain hash
  bytes32 private constant EIP712_DOMAIN_TYPEHASH = keccak256(abi.encodePacked(EIP712_DOMAIN));

  /// @notice address denoting native ether
  address private constant NATIVE_TOKEN = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;

  /// @notice EIP712 name
  bytes32 private constant EIP_712_NAME = keccak256("EthenaMinting");

  /// @notice holds EIP712 revision
  bytes32 private constant EIP712_REVISION = keccak256("1");
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L28-L58


```solidity
File:   contracts/StakedUSDe.sol
  bytes32 private constant REWARDER_ROLE = keccak256("REWARDER_ROLE");
  /// @notice The role that is allowed to blacklist and un-blacklist addresses
  bytes32 private constant BLACKLIST_MANAGER_ROLE = keccak256("BLACKLIST_MANAGER_ROLE");
  /// @notice The role which prevents an address to stake
  bytes32 private constant SOFT_RESTRICTED_STAKER_ROLE = keccak256("SOFT_RESTRICTED_STAKER_ROLE");
  /// @notice The role which prevents an address to transfer, stake, or unstake. The owner of the contract can redirect address staking balance if an address is in full restricting mode.
  bytes32 private constant FULL_RESTRICTED_STAKER_ROLE = keccak256("FULL_RESTRICTED_STAKER_ROLE");
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L26-L32

 14 instances of this in above






## [G‑09] The result of function calls should be cached rather than re-calling the function

Caching the result of function calls instead of repeatedly calling the function can lead to gas savings in Solidity.


```solidity
File:  contracts/StakedUSDe.sol
90   if (getUnvestedAmount() > 0) revert StillVesting();
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L90-L91



## [G-10] Can Make The Variable Outside The Loop To Save Gas 

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.



```solidity
File:  contracts/EthenaMinting.sol
425   uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L425



## [G-11] Duplicated require()/if() checks should be refactored to a modifier or function

Refactoring duplicated require() or if() checks into a modifier or a function can lead to gas savings in Solidity. When multiple conditions need to be checked at different points in a contract, consolidating the checks into a reusable modifier or function helps eliminate code duplication and reduce gas costs.


```solidity
File:  contracts/EthenaMinting.sol
172  if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L172

same condation  203 line

## [G‑12] Add unchecked {} for subtractions where the operands cannot underflow

Using unchecked {} blocks for subtractions where the operands cannot underflow can lead to gas savings in Solidity. 

```solidity
File:  contracts/EthenaMinting.sol
429   uint256 remainingBalance = amount - totalTransferred;
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L429