## Low

### [L-1] Functions permissions aren't the same as documented

In the contract `StakedUSDe.sol` the functions [`addToBlacklist()`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L106) and [`removeFromBlacklist()`](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L120) both say in the NatSpec comments that the owner and the blacklist manager can call these functions. However the code only allows the blacklist manager to call these functions.

## Non-Critical Issues

### [N-1] Misleading modifier names

In `SingleAdminAccessControl.sol` the name of the [modifier `notAdmin()`](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/SingleAdminAccessControl.sol#L17) should be more specific (i.e. `roleNotAdmin()`). The current name is too generic, leading to confusion and could be interpreted as what is being check is if the `msg.sender` is the `admin` (as often seen in other protocols).

The same issue appears in the `StakedUSDe.sol` contract, where a modifier with a similar name is used here [(`notOwner()`)](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L56), but now what is being checked is if the blacklist target is not the owner. Thus, a more clear modifier name could be `blacklistTargetNotOwner()`.

### [N-2] Misleading error message

In `SingelAdminAccessControl.sol`, in [line 18](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/SingleAdminAccessControl.sol#L18), the error message does not match what is happening in the modifier. This modifier checks if a given `role` is the `DEFAULT_ADMIN_ROLE`, and thus the error `InvalidAdminChange` should be replaced with a more suitable error such as `RoleIsAdmin`.

### [N-3] Use of wrong interface

In `USDeSilo.sol` when establishing the `USDe` contract [here](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDeSilo.sol#L20), the `IERC20` interface is used. This is fine since for now the only function called in the contract is the transfer function that is in the `IERC20` interface. However, since there is a `IUSDe` interface already written that extends `IERC20` it should be used here for clarity purposes and for future uses.