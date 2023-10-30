# ETHENA LABS  - QA REPORT.

| ID    |   Title    |
|:---:  |:---          |
| 1.  | Use Named Parameters in Mapping Types to Improve Code Readability. |
| 2.  | Use the Same Compiler Version in the Whole CodeBase.  |
| 3.  | Follow the Recommended Solidity Layout in Contracts to Improve Code Readability.  |
| 4.  | Instead of Hashing in the Contract Calculate the Hash off-chain. |



### 1.- Use Named Parameters in Mapping Types to Improve Code Readability.

Since solidity 0.8.18 named parameters in mappings types are allowed, this feature was enabled to improve code readability, this is why it's recommended as a good practice to use named parameters in mapping to improve code readability for your team members or other developers like auditors.

you can implement this feature in all mappings, for example in the mintedPerBlock mapping in the EthenaMining contract:
```solidity
/// @notice USDe minted per block
mapping(uint256 => uint256) public mintedPerBlock;
```
you can improve the readability of the mapping declaration like this:
```solidity
/// @notice USDe minted per block
mapping(uint256 blockNumber => uint256 amountMinted) public mintedPerBlock;
```
These are the Github links to the mappings in the codebase on which you can implement this improvement:

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L77-L87

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L381

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L393

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L18



### 2.- Use the Same Compiler Version in the Whole CodeBase.

looking at the files in the repository I found that almost all contracts are using the fixed solidity version 0.8.19, except for the USDeSilo contract which uses the floating pragma ^0.8.0.

 For security, it’s recommended to use the same fixed compiler pragma version in all your contracts, in this way, you can know with certainty the compiler version all your contracts use, so you know which compiler bugs can affect your contracts, and you only have to worry for the bug of one compiler version for all your contracts.

it’s recommended to use the fixed pragma solidity version in USDeSilo.sol.
Change the current pragma from this:
```solidity
pragma solidity ^0.8.0;
```
To this:
```solidity
pragma solidity 0.8.19;
```
This is the GitHub link to the code section:
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDeSilo.sol#L2


### 3.- Follow the Recommended Solidity Layout in Contracts to Improve Code Readability.

Solidity documentation recommends following a specified layout in contract declarations to improve code readability, this is a good practice recommended for writing clean, and easy-to-maintain code.

The USDeSilo contract doesn’t follow the recommended solidity layout, which says the following:
 https://docs.soliditylang.org/en/latest/style-guide.html#order-of-layout

Inside each contract, library or interface, use the following order:
Type declarations
State variables
Events
Errors
Modifiers
Functions
Just as a reminder, solidity also recommends a way to order the functions in the contracts:

https://docs.soliditylang.org/en/latest/style-guide.html#order-of-functions


Ordering helps readers identify which functions they can call and find the constructor and fallback definitions more easily.

Functions should be grouped according to their visibility and ordered:
constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
Within a grouping, place the view and pure functions last.
example:
```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity 0.8.19;
contract A {
    constructor() {
        // ...
    }

    receive() external payable {
        // ...
    }

    fallback() external {
        // ...
    }

    // External functions
    // ...

    // External functions that are view
    // ...

    // External functions that are pure
    // ...

    // Public functions
    // ...

    // Internal functions
    // ...

    // Private functions
    // ...
}

```

### 4.- Instead of Hashing in the Contract Calculate the Hash off-chain. 

The contracts EthenaMinting and StakeUSDe are using the keccak256 function to calculate the hash for roles and functions signatures assigned to constant values during deployment, this increases the deployment gas cost.

In order to improve gas efficiency and reduce gas costs during the contracts deployment, you can hash the values off-chain in a tool like https://emn178.github.io/online-tools/keccak_256.html  and just assign the calculated hash of the values to the constant declared in the contract.

for example, in the StakeUSDe you can hash the value of the BLACKLIST_MANAGER_ROLE in the tool mentioned before and assign that value in the contract.
 
You can change this:

```solidity
/// @notice The role that is allowed to blacklist and un-blacklist addresses
bytes32 private constant BLACKLIST_MANAGER_ROLE = keccak256("BLACKLIST_MANAGER_ROLE");
```
To this:
```solidity
/// @notice The role that is allowed to blacklist and un-blacklist addresses
bytes32 private constant BLACKLIST_MANAGER_ROLE = 0xf988e4fb62b8e14f4820fed03192306ddf4d7dbfa215595ba1c6ba4b76b369ee;
```
and you can do something similar with the rest of the constant values that use keccak256.

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L25-L32

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L27-L46

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L54-L58
