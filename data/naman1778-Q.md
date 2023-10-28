## [N-01] Floating pragma

Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.
https://swcregistry.io/docs/SWC-103

There is 1 instance of this issue in 1 file:

    File: USDeSilo.sol	

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.0;

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol

## [N-02] Lack of address(0) checks in the constructor

Zero-address check should be used in the constructors, to avoid the risk of setting smth as address(0) at deploying time.

There are 2 instances of this issue in 2 files:

    File: StakedUSDeV2.sol	

    42: constructor(IERC20 _asset, address initialRewarder, address owner) StakedUSDe(_asset, initialRewarder, owner) {

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol

    File: USDeSilo.sol	

    18: constructor(address stakingVault, address usde) {

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol

## [N-03] According to the syntax rules, use *=> mapping (* instead of *=> mapping(* using spaces as keyword

There are 2 instances of this issue in 1 file:

    File: EthenaMinting.sol

    78: mapping(address => mapping(uint256 => uint256)) private _orderBitmaps;

    86: mapping(address => mapping(address => bool)) public delegatedSigner;

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol

## [N-04] Inconsistent Solidity Versions

All the contracts except the below mentioned contracts use 0.8.19 version of solidity:

There is 1 instance of this issue in 1 file:

    File: USDeSilo.sol	

    2: pragma solidity ^0.8.0;

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol

## [N-05] Empty/Unused Function Parameters

Empty or unused function parameters should be commented out as a better and declarative way to silence runtime warning messages. As an example, the following function may have these parameters refactored to:

There are 2 instances of this issue in 1 file:

    File: StakedUSDeV2.sol	

    245: function _beforeTokenTransfer(address from, address to, uint256) internal virtual override {

    257: function renounceRole(bytes32, address) public virtual override {

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol