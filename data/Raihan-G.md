# GAS OPTIMIZATION

# SUMMARY
|      |  issue  |  instance  |
|------|---------|------------|
|[G‑01]|Avoid contract existence checks by using low level calls|5|
|[G‑02]|Can Make The Variable Outside The Loop To Save Gas |1|
|[G‑03]|Use do while loops instead of for loops|2|
|[G‑04]|A modifier used only once and not being inherited should be inlined to save gas|3|
|[G‑05]|Expressions for constant values such as a call to keccak256(), should use immutable rather than constant|14|
|[G‑06]|Avoid updating storage when the value hasn't changed|4|
|[G‑07]|Amounts should be checked for 0 before calling a transfer|6|
|[G‑08]|Avoid emitting storage values|6|
|[G‑09]|Delete variables that you don’t need|1|
|[G‑10]|With assembly, .call (bool success) transfer can be done gas-optimized|2|
|[G‑11]|Use Assembly To Check For address(0)|3|
|[G‑12]|Duplicated require()/if() checks should be refactored to a modifier or function|1|
|[G‑13]|Add unchecked {} for subtractions where the operands cannot underflow|1|
|[G‑14]|Unnecessary computation|2|
|[G‑15]|`keccak256()` should only need to be called on a specific string literal once|3|
|[G‑16]|The result of function calls should be cached rather than re-calling the function|1|
|[G‑17]|abi.encode() is less efficient than abi.encodePacked()|1|
|[G‑18]|Use Modifiers Instead of Functions To Save Gas|1|
|[G‑19]|Use assembly to validate msg.sender|4|
|[G‑20]|Use `msg.sender` instead of OpenZeppelin's `_msgSender()` when meta-transactions capabilities aren't used|2|
|[G‑21]|Refector function tos save gas|1|
|[G‑22]|Common math operations, like min and max have gas efficient alternatives|~|
|[G‑23]|Split revert statements|6|
|[G‑24]|Write gas-optimal for-loops|~|
|[G-25]|Don’t make variables public unless it is necessary to do so|~|


## [G‑01] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

There are 5 instances of this issue:

```solidity
File: contracts/EthenaMinting.sol#L253
253   IERC20(asset).safeTransfer(wallet, amount);

408   IERC20(asset).safeTransfer(beneficiary, amount);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L253



```solidity
File:  contracts/StakedUSDe.sol
96   IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L96


```solidity
File:  contracts/StakedUSDe.sol
140   IERC20(token).safeTransfer(to, amount);

167   return IERC20(asset()).balanceOf(address(this)) - getUnvestedAmount();
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L140

## [G-02] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

There are 1 instances of this issue:


```solidity
File:  contracts/EthenaMinting.sol
425   uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L425


## [G-03] Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration.
[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-09-use-do-while-loops-instead-of-for-loops)

There are 2 instances of this issue:


```solidity
File:  contracts/EthenaMinting.sol
120 if (_assets.length == 0) revert NoAssetsProvided();
121 if (_admin == address(0)) revert InvalidZeroAddress();
122 usde = _usde;
123
124 _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
125
126 for (uint256 i = 0; i < _assets.length; i++) {




360 if (route.addresses.length == 0) {
361   return false;
362 }
363 for (uint256 i = 0; i < route.addresses.length; ++i) {        
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L120-L126

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L360-L363


## [G-04] A modifier used only once and not being inherited should be inlined to save gas
When you use a modifier in Solidity, Solidity generates code to check the conditions of the modifier and execute the modified function if the conditions are met. This generated code can consume gas, especially if the modifier is used frequently or if the modified function is called multiple times.
By inlining a modifier that is used only once and not being inherited, you can eliminate the overhead of the generated code and reduce the gas cost of your contract.

There are 2 instances of this issue:
```solidity
File:  contracts/EthenaMinting.sol
97   modifier belowMaxMintPerBlock(uint256 mintAmount) {

104  modifier belowMaxRedeemPerBlock(uint256 redeemAmount) {    
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L97


## [G-05] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

The reason for this is that constant variables are evaluated at runtime and their value is included in the bytecode of the contract. This means that any expensive operations performed as part of the constant expression, such as a call to keccak256(), will be executed every time the contract is deployed, even if the result is always the same. This can result in higher gas costs.

There are 14 instances of this issue:
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

## [G‑06] Avoid updating storage when the value hasn't changed
If the old value is equal to the new value, not re-storing the value will avoid a Gsreset (2900 gas), potentially at the expense of a Gcoldsload (2100 gas) or a Gwarmaccess (100 gas)

There are 4 instances of this issue:
```solidity
File:  contracts/EthenaMinting.sol
438   maxMintPerBlock = _maxMintPerBlock;

445   maxRedeemPerBlock = _maxRedeemPerBlock;
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L438


```solidity
File:  contracts/StakedUSDe.sol
93  vestingAmount = newVestingAmount;
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L93

```solidity
File:  contracts/StakedUSDeV2.sol
132  cooldownDuration = duration;
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L132


## [G-07] Amounts should be checked for 0 before calling a transfer
Checking non-zero transfer values can avoid an expensive external call and save gas.

There are 6 instances of this issue:
```solidity
File:  contracts/EthenaMinting.sol
253   IERC20(asset).safeTransfer(wallet, amount);

408   IERC20(asset).safeTransfer(beneficiary, amount);

426   token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L253


```solidity
File:  contracts/StakedUSDe.sol
96    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

140   IERC20(token).safeTransfer(to, amount);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L140


```solidity
File:  contracts/USDeSilo.sol
29    USDE.transfer(to, amount);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L29




## [G-08] Avoid emitting storage values
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.

There are 6 instances of this issue:
```solidity
File:  contracts/EthenaMinting.sol
439    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);

446    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L439


```solidity
File:  contracts/SingleAdminAccessControl.sol
28    emit AdminTransferRequested(_currentDefaultAdmin, newAdmin);

74    emit AdminTransferred(_currentDefaultAdmin, account);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol#L28


```solidity
File:  contracts/StakedUSDeV2.sol
133  emit CooldownDurationUpdated(previousDuration, cooldownDuration);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L133


```solidity
File:  contracts/USDe.sol
24   emit MinterUpdated(newMinter, minter);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L24

## [G-09] Delete variables that you don’t need
In Ethereum, you get a gas refund for freeing up storage space.
Deleting a variable refund 15,000 gas up to a maximum of half the gas cost of the transaction. Deleting with the delete keyword is equivalent to assigning the initial value for the data type, such as 0 for integers.

There are 1 instances of this issue:
```solidity
File:  contracts/SingleAdminAccessControl.sol
77  delete _pendingDefaultAdmin;
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol#L77


## [G-10] With assembly, .call (bool success) transfer can be done gas-optimized
return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.
[Reffrence](https://code4rena.com/reports/2023-01-biconomy#g-01-with-assembly-call-bool-success-transfer-can-be-done-gas-optimized)

There are 2 instances of this issue:
```solidity
File:  contracts/EthenaMinting.sol
250     (bool success,) = wallet.call{value: amount}("");

404     (bool success,) = (beneficiary).call{value: amount}("");
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L250


## [G-11] Use Assembly To Check For address(0)
Saves 6 gas per instance if using assembly to check for address(0)
e.g.
```solidity
assembly {
 if iszero(_addr) {
  mstore(0x00, "zero address")
  revert(0x00, 0x20)
 }
}
```
There are 3 instances of this issue:

```solidity
File: contracts/EthenaMinting.sol
121  if (_admin == address(0)) revert InvalidZeroAddress();

343  if (order.beneficiary == address(0)) revert InvalidAmount();
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L121


```solidity
File:  contracts/USDe.sol
19   if (admin == address(0)) revert ZeroAddressException();
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L19

## [G-12] Duplicated require()/if() checks should be refactored to a modifier or function
sing modifiers or functions can make your code more gas-efficient by reducing the overall number of operations that need to be executed. For example, if you have a complex validation check that involves multiple operations, and you refactor it into a function, then the function can be executed with a single opcode, rather than having to execute each operation separately in multiple locations.

There are 1 instances of this issue:
```solidity
File:  contracts/EthenaMinting.sol
// @audit same condation in line 203
172  if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L172

## [G‑13] Add unchecked {} for subtractions where the operands cannot underflow

There are 1 instances of this issue:
```solidity
File:  contracts/EthenaMinting.sol
429   uint256 remainingBalance = amount - totalTransferred;
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L429


## [G-14] Unnecessary computation
When emitting an event that includes a new and an old value, it is cheaper in gas to avoid caching the old value in memory. Instead, emit the event, then save the new value in storage.

Proof of Concept
Instances include:

```
OwnableProxyDelegation.sol
function _setOwner
Recommended Mitigation
```
Replace
```
address oldOwner = _owner;
_owner = newOwner;
emit OwnershipTransferred(oldOwner, newOwner)
```
with
```
emit OwnershipTransferred(_owner_, newOwner)
_owner = newOwner;
```

There are 2 instances of this issue:
```solidity
File:  contracts/EthenaMinting.sol
    uint256 oldMaxMintPerBlock = maxMintPerBlock;
    maxMintPerBlock = _maxMintPerBlock;
    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L433-L439


```solidity
File:  contracts/EthenaMinting.sol
    uint256 oldMaxRedeemPerBlock = maxRedeemPerBlock;
    maxRedeemPerBlock = _maxRedeemPerBlock;
    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L444-L446


## [G-15] `keccak256()` should only need to be called on a specific string literal once
It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to `bytes4` should also only be done once


There are 3 instances of this issue:
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
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L28-L37


## [G‑16] The result of function calls should be cached rather than re-calling the function

There are 1 instances of this issue:
```solidity
File:  contracts/StakedUSDe.sol
// @audit getUnvestedAmount() fucntion call also use in line 91
90   if (getUnvestedAmount() > 0) revert StillVesting();
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L90-L91

## [G-17] abi.encode() is less efficient than abi.encodePacked()
In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

There are 1 instances of this issue:
```solidity
File:  contracts/EthenaMinting.sol
// @audit if you chnage abi.encode to abi.encodePacked there is no chance of collision but abi.encodePacked is gas-efficient
452   return keccak256(abi.encode(EIP712_DOMAIN, EIP_712_NAME, EIP712_REVISION, block.chainid, address(this)));
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L452



## [G-18] Use Modifiers Instead of Functions To Save Gas
Example of two contracts with modifiers and internal view function:
```
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
Differences:
```
Deploy Modifier.sol
108727
Deploy Inlined.sol
110473
Modifier.foo
21532
Inlined.foo
21556
```
This with 0.8.9 compiler and optimization enabled. As you can see it's cheaper to deploy with a modifier, and it will save you about 30 gas. But sometimes modifiers increase code size of the contract.


There are 1 instances of this issue:

```solidity
File:  contracts/EthenaMinting.sol
  /// @notice Checks if an asset is supported.
265  function isSupportedAsset(address asset) external view returns (bool) {
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L265-L267


## [G-19] Use assembly to validate msg.sender
[Reffrence](https://solodit.xyz/issues/g-06-use-assembly-to-validate-msgsender-code4rena-juicebox-juicebox-buyback-delegate-git)

There are 4 instances of this issue:
```solidity
File:  contracts/EthenaMinting.sol
138   if (msg.sender != _admin) {
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L138


```solidity
File:  contracts/SingleAdminAccessControl.sol
26   if (newAdmin == msg.sender) revert InvalidAdminChange();

32   if (msg.sender != _pendingDefaultAdmin) revert NotPendingAdmin();
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol#L26


```solidity
File:  contracts/USDe.sol
29   if (msg.sender != minter) revert OnlyMinter();
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L29


## [G-20] Use `msg.sender` instead of OpenZeppelin's `_msgSender()` when meta-transactions capabilities aren't used
msg.sender costs 2 gas (CALLER opcode). _msgSender() represents the following:
```
function _msgSender() internal view virtual returns (address payable) {
  return msg.sender;
}
```
When no meta-transactions capabilities are used: msg.sender is enough.

See [https://docs.openzeppelin.com/contracts/2.x/gsn](https://docs.openzeppelin.com/contracts/2.x/gsn) for more information about GSN capabilities.

[Reffrence](https://solodit.xyz/issues/g-15-use-code4rena-forgotten-runes-forgotten-runes-warrior-guild-contest-git)

There are 2 instances of this issue:

```solidity
File:  contracts/StakedUSDeV2.sol
103   _withdraw(_msgSender(), address(silo), owner, assets, shares);

119   _withdraw(_msgSender(), address(silo), owner, assets, shares);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L103-L119


## [G-21] Refector function tos save gas

There are 1 instances of this issue:
```solidity
File:  contracts/EthenaMinting.sol
377  function verifyNonce(address sender, uint256 nonce) public view override returns (bool, uint256, uint256, uint256) {
    if (nonce == 0) revert InvalidNonce();
    uint256 invalidatorSlot = uint64(nonce) >> 8;
    uint256 invalidatorBit = 1 << uint8(nonce);
    mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
    uint256 invalidator = invalidatorStorage[invalidatorSlot];
    if (invalidator & invalidatorBit != 0) revert InvalidNonce();

    return (true, invalidatorSlot, invalidator, invalidatorBit);
  }
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L377

`Fix code:`
```diff
function verifyNonce(address sender, uint256 nonce) public view override returns (bool, uint256, uint256, uint256) {
    if (nonce == 0) revert("InvalidNonce");

    uint256 invalidatorSlot = uint64(nonce) >> 8;
    uint256 invalidatorBit = 1 << uint8(nonce);
--  mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
--  uint256 invalidator = invalidatorStorage[invalidatorSlot];
++  uint256 invalidator = _orderBitmaps[sender][invalidatorSlot];

    if ((invalidator & invalidatorBit) != 0) revert("InvalidNonce");

    return (true, invalidatorSlot, invalidator, invalidatorBit);
}

```





## [G‑22] Common math operations, like min and max have gas efficient alternatives
Unoptimized
```
function max(uint256 x, uint256 y) public pure returns (uint256 z) {
	z = x > y ? x : y;
}
```
Optimized
```
function max(uint256 x, uint256 y) public pure returns (uint256 z) {
    /// @solidity memory-safe-assembly
    assembly {
        z := xor(x, mul(xor(x, y), gt(y, x)))
    }
}
```
The code above is taken from the math section of the Solady Library, more math operations can be found there. It is worth exploring the library to see what gas efficient operations are available to you.


The reason the above example is more gas efficient is because the ternary operator (and in general, code with conditionals in it) contain conditional jumps in the opcodes, which are more costly.

## [G-23] Split revert statements
Similar to splitting require statements, you will usually save some gas by not having a boolean operator in the if statement.

```
contract CustomErrorBoolLessEfficient {
    error BadValue();

    function requireGood(uint256 x) external pure {
        if (x < 10 || x > 20) {
            revert BadValue();
        }
    }
}
```

```
contract CustomErrorBoolEfficient {
    error TooLow();
    error TooHigh();

    function requireGood(uint256 x) external pure {
        if (x < 10) {
            revert TooLow();
        }
        if (x > 20) {
            revert TooHigh();
        }
    }
}
```

There are 6 instances of this issue:
```solidity
File:  contracts/EthenaMinting.sol
248   if (wallet == address(0) || !_custodianAddresses.contains(wallet)) revert InvalidAddress();

291   if (asset == address(0) || asset == address(usde) || !_supportedAssets.add(asset)) {
      revert InvalidAssetAddress();
    }

299 if (custodian == address(0) || custodian == address(usde) || !_custodianAddresses.add(custodian)) {
      revert InvalidCustodianAddress();
    }   

421   if (!_supportedAssets.contains(asset) || asset == NATIVE_TOKEN) revert UnsupportedAsset();     
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L248


```solidity
File:  contracts/StakedUSDe.sol
75    if (_owner == address(0) || _initialRewarder == address(0) || address(_asset) == address(0)) {
      revert InvalidZeroAddress();
    }

210    if (hasRole(SOFT_RESTRICTED_STAKER_ROLE, caller) || hasRole(SOFT_RESTRICTED_STAKER_ROLE, receiver)) {
      revert OperationNotAllowed();
    }    
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L75


## [G-24] Write gas-optimal for-loops
This is what a gas-optimal for loop looks like, if you combine the two tricks above:

for (uint256 i; i < limit; ) {
    
    // inside the loop
    
    unchecked {
        ++i;
    }
}

The two differences here from a conventional for loop is that i++ becomes ++i (as noted above), and it is unchecked because the limit variable ensures it won’t overflow.


## [G-25] Don’t make variables public unless it is necessary to do so
A public storage variable has an implicit public function of the same name. A public function increases the size of the jump table and adds bytecode to read the variable in question. That makes the contract larger.

Remember, private variables aren’t private, it’s not difficult to extract the variable value using web3.js.

This is especially true for constants which are meant to be read by humans rather than smart contracts.