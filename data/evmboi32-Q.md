## MORE UPDATED VERSION OF SOLIDITY CAN BE USED

Using the more updated version of Solidity can add new features and enhance security. As described in https://github.com/ethereum/solidity/releases, Version 0.8.22 is the latest version of Solidity. Please consider using the more updated version of Solidity for all contracts.

## HARDCODED STRING THAT IS REPEATEDLY USED CAN BE REPLACED WITH A CONSTANT
The number ```10_000``` is used twice in the ```EthenaMinting.sol```

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L370-L372

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L425

For better maintainability, please consider creating and using a constant for ```10_000``` instead of hardcoding  ```10_000```.

## NO SAME VALUE INPUT CONTROL
Check if the new value is the same as the old value.

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L436-L440

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L443-L447

```solidity
  /// @notice Sets the max mintPerBlock limit
  function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {
    uint256 oldMaxMintPerBlock = maxMintPerBlock;
    maxMintPerBlock = _maxMintPerBlock;
    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);
  }

  /// @notice Sets the max redeemPerBlock limit
  function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {
    uint256 oldMaxRedeemPerBlock = maxRedeemPerBlock;
    maxRedeemPerBlock = _maxRedeemPerBlock;
    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);
  }

```

## According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L18

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L78-L86

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L381

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L393

```solidity
StakedUSDeV2.sol
  mapping(address => UserCooldown) public cooldowns;
  
EthenaMinting.sol
  mapping(address => mapping(uint256 => uint256)) private _orderBitmaps;
  mapping(uint256 => uint256) public mintedPerBlock;
  mapping(uint256 => uint256) public redeemedPerBlock;
  mapping(address => mapping(address => bool)) public delegatedSigner;
  mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
  mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
```

## Use SMTChecker
The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## Declaration shadows an existing declaration
The following variables shadows the ```owner``` variable form ```SingleAdminAccessControl.sol```
``` solidity
function cooldownAssets(uint256 assets, address owner)
function cooldownShares(uint256 shares, address owner) 
function withdraw(uint256 assets, address receiver, address owner)
function redeem(uint256 shares, address receiver, address owner)
```

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L95

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L111

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L52

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L65