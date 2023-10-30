| ID                | Title                                                                        |
| ----------------- | ---------------------------------------------------------------------------- |
| [GAS-01](#GAS-01) | Prefer `abi.encodePacked()` over `abi.encode()`                              |
| [GAS-02](#GAS-02) | Use `bytes.concat()` Instead of `abi.encodePacked`                           |
| [GAS-03](#GAS-03) | Emit Memory Values Instead of Storage Values                                 |
| [GAS-04](#GAS-04) | Use assembly for math equations                                              |
| [GAS-05](#GAS-05) | Use assembly to validate `msg.sender`                                        |
| [GAS-06](#GAS-06) | Use the inputs/results of assignments rather than re-reading state variables |
| [GAS-07](#GAS-07) | x + y is more efficient than using += for state variables (likewise for -=)  |

##
---
##

## [GAS-01] Prefer `abi.encodePacked()` over `abi.encode()`

For more information, refer to: [Solidity Encode Gas Comparison](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)

**Issue Description:**
There is one instance of this issue in the `EthenaMinting.sol` contract:

```solidity
📁 File: protocols/USDe/contracts/EthenaMinting.sol

49:   bytes32 private constant EIP712_DOMAIN_TYPEHASH = keccak256(abi.encodePacked(EIP712_DOMAIN)); 
```
[49](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol/#L49)

---

## [GAS-02] Use `bytes.concat()` Instead of `abi.encodePacked`

When concatenation is not used for hashing, `bytes.concat` is the preferred method as it is more gas-efficient.

**Issue Description:**
There is one instance of this issue in the `EthenaMinting.sol` contract:

```solidity
📁 File: protocols/USDe/contracts/EthenaMinting.sol

49:   bytes32 private constant EIP712_DOMAIN_TYPEHASH = keccak256(abi.encodePacked(EIP712_DOMAIN)); 
```
[49](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol/#L49-L49)

**Recommended Mitigation Steps:**
Consider using `bytes.concat()` and upgrading to at least Solidity version 0.8.4 if required.

---

## [GAS-03] Emit Memory Values Instead of Storage Values

Emit memory values instead of reading storage values again. This optimization can result in significant gas savings.

**Gas Saved per Instance:** ~97 *(Total: ~582)*

**Issue Description:**
There are six instances of this issue:

```solidity
📁 File: protocols/USDe/contracts/EthenaMinting.sol

439:     emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock); 

446:     emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock); 
```
[439](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol/#L439-L439), [446](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol/#L446-L446)

```solidity
📁 File: protocols/USDe/contracts/SingleAdminAccessControl.sol

28:     emit AdminTransferRequested(_currentDefaultAdmin, newAdmin); 

74:     emit AdminTransferred(_currentDefaultAdmin, account); 
```
[28](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol/#L28-L28), [74](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol/#L74-L74)

```solidity
📁 File: protocols/USDe/contracts/StakedUSDeV2.sol

133:     emit CooldownDurationUpdated(previousDuration, cooldownDuration); 
```
[133](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol/#L133-L133)

```solidity
📁 File: protocols/USDe/contracts/USDe.sol

24:     emit MinterUpdated(newMinter, minter); 
```
[24](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol/#L24-L24)

Emitting memory values instead of reading storage values again can result in gas savings.


## [GAS-04]: Use assembly for math equations

Replace `(a * b) / c` with the assembly equivalent `div(mul(a, b), c)` to save gas.

#### Example mitigation:

```solidity
// Replace (a * b) / c with assembly
uint256 result;
assembly {
    result := div(mul(a, b), c)
}
```

## [GAS-05]: Use assembly to validate `msg.sender`

You can use assembly to efficiently validate `msg.sender`. Here's an example of how to do this:

#### Example mitigation:

```solidity
// Use assembly to validate msg.sender
assembly {
    if iszero(eq(msg.sender, _admin)) {
        revert(0, 0)
    }
}
```

## [GAS-06]: Use the inputs/results of assignments rather than re-reading state variables

You should use the result of assignments rather than re-reading state variables. Here's an example:

#### Example mitigation:

```solidity
// Use the result of the assignment instead of re-reading the state variable
uint256 newMaxMintPerBlock = maxMintPerBlock;
maxMintPerBlock = newMaxMintPerBlock + increment;
emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);
```

## [GAS-07]: x + y is more efficient than using += for state variables (likewise for -=)

You should use `x = x + y` instead of `x += y` for state variables to save gas. Here's an example:

#### Example mitigation:

```solidity
// Use x = x + y instead of x += y
mintedPerBlock[block.number] = mintedPerBlock[block.number] + order.usde_amount;
```