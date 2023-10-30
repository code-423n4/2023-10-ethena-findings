### [GAS-1] Add internal functions to remove extra logic.

In the constructor of `EthenaMinting.sol` the role of `DEFAULT_ADMIN_ROLE` is first given to [`msg.sender`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L124) and then to [`_admin`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L139). This is done in the constructor so the functions [`addSupportedAsset()`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L290) and [`addCustodianAddress()`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L298) can be called since they have the condition `onlyRole(DEFAULT_ADMIN_ROLE)`.
The contract can be optimized by creating internal functions for adding the assets and custodian addresses that do not have the modifier, which will be called in the constructor and in the public add functions which wil keep the modifier. This way in the constructor you only need to call `_grantRole()` once for the `_admin` without the `if` condition saving on gas.

### [GAS-2] Remove `return` value since it will always be the same.

In the contract `EthenaMinting.sol` the functions [`VerifyOrder()`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L347) and [`verifyNonce()`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L385) return a boolean along other variables. This boolean will always be true so it is not needed. If the call doesn't revert we know that the verification worked so the boolean is unnecessary.

### [GAS-3] Unnecessary parameter and verification.

In the contract `EthenaMinting.sol` the function `VerifyRoute()` receives the [`orderType`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L351) and proceeds to [check](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L353) if it is a `REDEEM` if so it returns true. This function is only called internally from the `mint()` function, and if someone calls the function publicly they wouldn't say it is for a `REDEEM` as the route is only needed for the mint. Therefore there is no need to receive the `orderType` in the function nor is its verification.

### [GAS-4] Unnecessary creation of storage variable in  `VerifyNonce()`

In the contract `EthenaMinting.sol` the function `VerifyNonce()` creates the variable `invalidatorStorage` unnecessarily. It would be less gas to simply replace [lines 381 and 382](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L381-L382) with `uint256 invalidator = _orderBitmaps[sender][invalidatorSlot];`

### [GAS-5] Unnecessary creation of storage variable in  `_deduplicateOrder()`

In the contract `EthenaMinting.sol` the function `_deduplicateOrder()` creates the variable `invalidatorStorage` unnecessarily. It would be less gas to simply replace the [lines 393 and 394](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L393-L394) with `_orderBitmaps[sender][invalidatorSlot] = invalidator | invalidatorBit;`.

### [GAS-6] Adding a variable that will always be zero.

In the contract `StakedUSDe.sol` the function `transferInRewards()` checks that `getUnvestedAmount()` [returns 0](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L90). In the [next line](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L91) it calls `getUnvestedAmount()` again to add it to a variable, but we already know it will return 0. Remove the second call to `getUnvestedAmount()`.