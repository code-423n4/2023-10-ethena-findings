# Low-Level and Non-Critical issues

## Summary

### Low Level List

| Number | Issue Details                                                                             | Instances |
| :----: | :---------------------------------------------------------------------------------------- | :-------: |
| [L-01] | Check Low Level Calls for address 0  |   1   |
| [L-02] | Check receiver address is not address(0) and amount is greater than zero before transferring or minting any tokens  |   2   |

Total 2 Low Level Issues

### Non Critical List

| Number  | Issue Details                                                       | Instances |
| :-----: | :------------------------------------------------------------------ | :-------: |
| [NC-01] | `if (address(this).balance < amount) revert InvalidAmount();` this check should be applied before low level call for code clarity  |     1     |
| [NC-02] | Some Contracts are not following proper solidity style guide layout  |     3     |
| [NC-03] | Do not use inherited state variable/function name as function param name it will shadow the above and creates confusion  |     1     |

Total 3 Non Critical Issues

# Low-Level issues:

## [L-01]  Check Low Level Calls for address 0

### `beneficiary` can be address 0 so must be checked for address 0 before transferring ether to it.
If 0 passed as `beneficiary`  these ethers will be lost because no `address(0)` check is implemented before this  ether(or blockchain native token on other chain) transferring.
_1 Instance in 1 File_

```solidity
File: contracts/EthenaMinting.sol

404: (bool success,) = (beneficiary).call{value: amount}("");

```
[404](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L404)


## [L-02] Check receiver address is not address(0) and amount is greater than zero before transferring or minting any tokens

Check `to` is non 0 because some tokens do not revert when transferred to 0 address if they created using solmate erc20 file but they revert if created using openzeppelin erc20 and no token is specified here so token can be any erc20 token.

_2 Instances in 2 Files_

```solidity
File : contracts/StakedUSDe.sol

138: function rescueTokens(address token, uint256 amount, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {
139:    if (address(token) == asset()) revert InvalidToken();
140:    IERC20(token).safeTransfer(to, amount);
141:  }

```
[138-141](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L138C3-L141C4)

```solidity
File : contracts/USDeSilo.sol

28:  function withdraw(address to, uint256 amount) external onlyStakingVault {
29:    USDE.transfer(to, amount);
30:  }

```
[28-30](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L28C2-L30C4)


# Non Critical Issues:

## [NC-01] `if (address(this).balance < amount) revert InvalidAmount();` this check should be applied before low level call for code clarity

_1 Instance in 1 File_

```solidity
File: contracts/EthenaMinting.sol

function transferToCustody(...
  {...
     ...
    (bool success,) = wallet.call{value: amount}("");
  ...

```
[250](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L250)

## [NC-02] Some Contracts are not following proper solidity style guide layout

/ Layout of Contract: According to solidity Docs
// version
// imports
// errors
// interfaces, libraries, contracts
// Type declarations
// State variables
// Events
// Modifiers
// Functions

// Layout of Functions:
// constructor
// receive function (if exists)
// fallback function (if exists)
// external
// public
// internal
// private
// internal & private view & pure functions
// external & public view & pure functions
https://docs.soliditylang.org/en/latest/style-guide.html#order-of-layout

**Note: These instance missed by bot report**

_3 Instances in 2 File_

```solidity 
File : contracts/SingleAdminAccessControl.sol

  ///@audit declare view functions in last

65: function owner() public view virtual returns (address) {

```
[65](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol#L65)

```solidity
File : contracts/StakedUSDe.sol

///@audit declare view functions in last
166: function totalAssets() public view override returns (uint256) {

///@audit declare view functions in last
173: function getUnvestedAmount() public view returns (uint256) {    

```
[166](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L166), [173](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L173)


## [NC-03] Do not use inherited state variable/function name as function param name it will shadow the above and creates confusion

_1 Instance in 1 File_

```solidity
File : contracts/StakedUSDeV2.sol

   ///@audit owner

42: constructor(IERC20 _asset, address initialRewarder, address owner) StakedUSDe(_asset, initialRewarder, owner) {

```
[42](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L42)