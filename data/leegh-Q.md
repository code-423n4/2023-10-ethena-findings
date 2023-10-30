# Low
1. "owner" is not allowed to add/remove black list.

Function "addToBlacklist" and "removeFromBlacklist" are limited to called by BLACKLIST_MANAGER_ROLE only. But the owner is also allowed to call them according to the function notice.
```solidity
101:  /**
102:   * @notice Allows the owner (DEFAULT_ADMIN_ROLE) and blacklist managers to blacklist addresses.
103:   * @param target The address to blacklist.
104:   * @param isFullBlacklisting Soft or full blacklisting level.
105:   */
106:  function addToBlacklist(address target, bool isFullBlacklisting)
107:    external
108:    onlyRole(BLACKLIST_MANAGER_ROLE)
109:    notOwner(target)
110:  {
111:    bytes32 role = isFullBlacklisting ? FULL_RESTRICTED_STAKER_ROLE : SOFT_RESTRICTED_STAKER_ROLE;
112:    _revokeRole(role, target);
113:  }

115:  /**
116:   * @notice Allows the owner (DEFAULT_ADMIN_ROLE) and blacklist managers to un-blacklist addresses.
117:   * @param target The address to un-blacklist.
118:   * @param isFullBlacklisting Soft or full blacklisting level.
119:   */
120:  function removeFromBlacklist(address target, bool isFullBlacklisting)
121:    external
122:    onlyRole(BLACKLIST_MANAGER_ROLE)
123:    notOwner(target)
124：  function rescueTokens(address token, uint256 amount, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {
125：    if (address(token) == asset()) revert InvalidToken();
126：    IERC20(token).safeTransfer(to, amount);
127：  }
```
| [Line #101-113](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L101-L113) | [Line #115-127](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L115-L127) | 


# Non-critical
1. Parameters and struct members names not mixedCase

```solidity
File: EthenaMinting.sol
36:    "Order(uint8 order_type,uint256 expiry,uint256 nonce,address benefactor,address
```
| [Line #36](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L36) | 
```solidity
File: IEthenaMinting.sol

20:    SignatureType signature_type;
21:    bytes signature_bytes;
30:    OrderType order_type;
35:    address collateral_asset;
36:    uint256 collateral_amount;
37:    uint256 usde_amount;
63:  function verifyRoute(Route calldata route, OrderType order_type) external view returns (bool);
```
| [Line #20](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/interfaces/IEthenaMinting.sol#L20) | [Line #21](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/interfaces/IEthenaMinting.sol#L21) | [Line #30](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/interfaces/IEthenaMinting.sol#L30) | [Line #35](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/interfaces/IEthenaMinting.sol#L35) | [Line #36](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/interfaces/IEthenaMinting.sol#L36) | [Line #37](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/interfaces/IEthenaMinting.sol#L37) | [Line #63](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/interfaces/IEthenaMinting.sol#L63) | 