1. Redundant `_checkMinShares` modifier

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L190C1-L194C4

`_checkMinShares` was designed to prevent a donation attack caused by having zero shares. However, in the OpenZeppelin ERC4626 code in version 4.9.0, this risk has already been mitigated by introducing virtual assets and shares (reference: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC4626.sol). Therefore, there is no need to add this modifier to check for it.


2. Redundant Check in the `verifyRoute()`

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L351C1-L355C6

The `verifyRoute()` function is only used and verified in the context of minting. Therefore, passing the `orderType` parameter and verifying it is unnecessary.