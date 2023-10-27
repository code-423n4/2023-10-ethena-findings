L01 - https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol#L29

Although silo contract is only used by the USDe token it is still better to use safeTransfer in case it'll be used with other assets (Not in automated findings)

L02 - https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L91

`getUnvestedAmount()` is always 0, can be removed

N01 - https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L96
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L112

It's probably better to use default OZ ERC4626 errors `ERC4626ExceededMaxWithdraw` and `ERC4626ExceededMaxRedeem`, to be more consistent

N02 - https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L121

`InvalidAdminAddress` instead of `InvalidZeroAddress` will be more informative

N03 - https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L425

`10_000`, better to use it as a constant 

N04 - https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L347

`verifyOrder` is always `true` unless it reverts, redundant return value