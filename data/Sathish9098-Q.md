# LOW Findings

## 

## [L-1] ``address(this).balance < amount`` this check can be bypassed by attacker to send some tokens to contract through front running 

The check address(this).balance < amount is used to ensure that the contract has a sufficient balance of Ether (ETH) to perform some operation, typically involving transferring ETH to another address. However, you are correct that this check can be bypassed by an attacker if they send some other tokens to the contract.

The address(this).balance only represents the Ether balance of the contract, and it doesn't take into account other tokens held by the contract. Therefore, an attacker can send a different ERC-20 token to the contract and potentially trick the contract into performing actions that it shouldn't. This is why it's crucial to be careful when dealing with the transfer of assets, especially if the contract makes assumptions about the asset being transferred

### POC

```solidity
FILE: 2023-10-ethena/contracts/EthenaMinting.sol

403: if (address(this).balance < amount) revert InvalidAmount();

```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L403

### Recommended Mitigation
Before transferring tokens, use the balanceOf function from the ERC-20 token contract to check the contract's balance for the specific token.

##

## [L-2] Hardcoded Chainid

Hardcoding the chain ID in your contract can have some security and operational implications. Chain IDs represent different blockchain networks, and hardcoding a specific chain ID can restrict your contract to only operate on a particular network. This can limit your contract's flexibility and portability. Additionally, it might introduce vulnerabilities if the contract is deployed on a different chain with a different ID

```solidity
FILE: 2023-10-ethena/contracts/EthenaMinting.sol

72:  uint256 private immutable _chainId;

```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L72C1-L72C38

##

## [L-3] Minter can manipulate the block.timestamp to cause the order to expire prematurely

the Minter can manipulate the block.timestamp to cause the order to expire prematurely. This is a potential security vulnerability

```solidity
FILE: 2023-10-ethena/contracts/EthenaMinting.sol

346: if (block.timestamp > order.expiry) revert SignatureExpired();

```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L346



