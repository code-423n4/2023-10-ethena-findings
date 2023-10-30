# Low

# Summary
|      |  issue  |  instance  |
|------|---------|------------|
|[L-01]|_safeMint() should be used rather than _mint() wherever possible|2|
|[L-02]|Contracts are not using their OZ upgradeable counterparts|3|
|[L-03]|Use `safetransfer` Instead Of `transfer`|1|
|[L-04]|Upgrade Open Zeppelin contract dependency|~|
|[L-05]|no check if the burn amount is zero or not|1|
|[L-06]|Missing Revert Condition for block.timestamp == order.expiry in verifyOrder function|1|
|[L-07]|Gas griefing/theft is possible on unsafe external call|2|
|[L-08]|Use fixed compiler version |1|
|[L-09]|Return values of transfer()/transferFrom() not checked|1|
|[L-10]|arbitrary send ERC20|3|
|[L-11]|Unprotected Ether Transfer to Arbitrary Address|2|
|[L-12]|Use `safeTransferOwnership` instead of `transferOwnership` function|1|

## [L-01] _safeMint() should be used rather than _mint() wherever possible
_mint() is [discouraged](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L271) in favor of _safeMint() which ensures that the recipient is either an EOA or implements IERC721Receiver. Both open [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L238-L250) and [solmate](https://github.com/Rari-Capital/solmate/blob/4eaf6b68202e36f67cab379768ac6be304c8ebde/src/tokens/ERC721.sol#L180) have versions of this function so that NFTs aren’t lost if they’re minted to contracts that cannot transfer them back out.


```solidity
File:  contracts/StakedUSDe.sol
153   if (to != address(0)) _mint(to, amountToDistribute);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L153


```solidity
File:  contracts/USDe.sol
30   _mint(to, amount);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L30

## [L-02] Contracts are not using their OZ upgradeable counterparts
Description
The non-upgradeable standard version of OpenZeppelin's library, such as `Ownable`, `Pausable`, `Address`, `Context`, `SafeERC20`, `ERC1967Upgrade` etc, are inherited / used by both the proxy and the implementation contracts.

As a result, when attempting to use the upgrades plugin mentioned, the following errors are raised:

```
Error: Contract `FiatTokenV1` is not upgrade safe

contracts/v1/FiatTokenV1.sol:58: Variable `totalSupply_` is assigned an initial value
  Move the assignment to the initializer
  https://zpl.in/upgrades/error-004

contracts/v1/Pausable.sol:49: Variable `paused` is assigned an initial value
  Move the assignment to the initializer
  https://zpl.in/upgrades/error-004

contracts/v1/Ownable.sol:28: Contract `Ownable` has a constructor
  Define an initializer instead
  https://zpl.in/upgrades/error-001

contracts/util/Address.sol:186: Use of delegatecall is not allowed
  https://zpl.in/upgrades/error-002
       
```
Having reviewed these errors, none had any adversarial impact:

`totalSupply_` and `paused` are explictly assigned the default values `0` and `false`
the implementation contracts utilises the internal `_transferOwnership()` in the initializer, thus transferring ownership to `newOwner` regardless of who the current owner is
`Address's` `delegatecall` is only used by the `ERC1967Upgrade` contract. Comparing both the `Address` and `ERC1967Upgrade` contracts against their upgradeable counterparts show similar behaviour (differences are some refactoring done to shift the delegatecall into the `ERC1967Upgrade` contract).
Nevertheless, it would be safer to use the upgradeable versions of the library contracts to avoid unexpected behaviour.


```solidity
File:  contracts/EthenaMinting.sol
10   import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L10


```solidity
File:  contracts/StakedUSDe.sol
9   import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L9


```solidity
File:  contracts/USDeSilo.sol
5     import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L5


### Recommended Mitigation Steps
Where applicable, use the contracts from `@openzeppelin/contracts-upgradeable` instead of `@openzeppelin/contracts`.


## [L-03] Use `safetransfer` Instead Of `transfer`
It is good to add a require() statement that checks the return value of token transfers or to use something like OpenZeppelin’s safeTransfer/safeTransferFrom unless one is sure the given token reverts in case of a failure. Failure to do so will cause silent failures of transfers and affect token accounting in contract.
For example, Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)'s transfer() and transferFrom() functions do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert.

```solidity
File:  contracts/USDeSilo.sol
29   USDE.transfer(to, amount);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L29

Recommended Mitigation Steps
Consider using safeTransfer/safeTransferFrom or require() consistently.


## [L-04] Upgrade Open Zeppelin contract dependency
An outdated OZ version is used (which has known vulnerabilities, see https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories).

Recommended Mitigation Steps
use new versions


## [L-05] no check if the burn amount is zero or not
if the amount is zero so the unhecked block will divided zero on 3 and we use gas for nothing ! if we set zero we may pass the _burn checks, i know it is passed by only permit but it's better to avoid this happen because it's seting by human and it means it can be set with 0 balance to burn !

```solidity
File:  contracts/StakedUSDe.sol
151   _burn(from, amountToDistribute);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L151


## [L-06] Missing Revert Condition for block.timestamp == order.expiry in verifyOrder function
The verifyOrder function is responsible for verifying the validity of an order by checking various conditions. However, there is a bug in the code where it fails to include a revert condition for the case when block.timestamp is equal to order.expiry. This omission can lead to unexpected behavior, as the condition will not revert in that specific scenario.


```solidity
File:  contracts/EthenaMinting.sol
346   if (block.timestamp > order.expiry) revert SignatureExpired();
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L346

`Recommendation`

```solidity
  if (block.timestamp >= order.expiry) revert SignatureExpired();
```

## [L-07] Gas griefing/theft is possible on unsafe external call
return data (bool success,) has to be stored due to EVM architecture, if in a usage like below, 'out' and 'outsize' values are given (0,0) . Thus, this storage disappears and may come from external contracts a possible Gas griefing/theft problem is avoided

```solidity
File:  contracts/EthenaMinting.sol
250     (bool success,) = wallet.call{value: amount}("");

404     (bool success,) = (beneficiary).call{value: amount}("");
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L250

## [L-08] Use fixed compiler version ([see](https://swcregistry.io/docs/SWC-103))

`USDeSilo` contract use `^0.8.0` as compiler version.

```solidity
File:  contracts/USDeSilo.sol
2     pragma solidity ^0.8.0;
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L2


## [L‑09] Return values of transfer()/transferFrom() not checked
Not all ERC20 implementations revert() when there's a failure in transfer() or transferFrom(). The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually transfer anything.

```solidity
File:  contracts/USDeSilo.sol
29    USDE.transfer(to, amount);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L29


## [L-10] arbitrary send ERC20
Detect when `msg.sender` is not used as `from` in transferFrom.

```solidity
File:  contracts/StakedUSDe.sol
96   IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

426   token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L96

```solidity
File:  contracts/StakedUSDe.sol
96    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L140

Recommendation:

Use `msg.sender` as `from` in transferFrom.


## [L-11] Unprotected Ether Transfer to Arbitrary Address
Description: The function transferToCustody in the EthenaMinting contract contains a call that sends Ether to an arbitrary address. This is potentially dangerous as it could allow unauthorized users to withdraw funds.

```solidity
File:  contracts/EthenaMinting.sol
250   (bool success,) = wallet.call{value: amount}("");

404   (bool success,) = (beneficiary).call{value: amount}("");
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L250

`Recommendation`: Implement access control mechanisms to ensure that only authorized users can execute this function. This could be achieved by using a modifier that checks if the caller has the necessary permissions. Additionally, consider using the OpenZeppelin's 'Address' library's 'sendValue' function to send Ether, which includes additional safety checks

## [L-02] Use `safeTransferOwnership` instead of `transferOwnership` function

```solidity
File:  contracts/USDe.sol
20   _transferOwnership(admin);
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L20