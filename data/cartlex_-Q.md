# [NC-01] DEFAULT_ADMIN_ROLE can't use `addToBlacklist`/`removeFromBlacklist` by default.

In the comments to `addToBlacklist` and `removeFromBlacklist` says that DEFAULT_ADMIN_ROLE can add users to blacklist and remove them from it. However, by default `admin` can't do it.

```solidity
/**
* @notice Allows the owner (DEFAULT_ADMIN_ROLE) and blacklist managers to blacklist addresses. 
* @param target The address to blacklist.
* @param isFullBlacklisting Soft or full blacklisting level.
*/
function addToBlacklist(address target, bool isFullBlacklisting) external onlyRole(BLACKLIST_MANAGER_ROLE) notOwner(target) {
    bytes32 role = isFullBlacklisting ? FULL_RESTRICTED_STAKER_ROLE : SOFT_RESTRICTED_STAKER_ROLE;
    _grantRole(role, target);
}
```
  https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L101-L113

```solidity
/**
 * @notice Allows the owner (DEFAULT_ADMIN_ROLE) and blacklist managers to un-blacklist addresses.
 * @param target The address to un-blacklist.
 * @param isFullBlacklisting Soft or full blacklisting level.
 */
function removeFromBlacklist(address target, bool isFullBlacklisting) external onlyRole(BLACKLIST_MANAGER_ROLE) notOwner(target) {
    bytes32 role = isFullBlacklisting ? FULL_RESTRICTED_STAKER_ROLE : SOFT_RESTRICTED_STAKER_ROLE;
    _revokeRole(role, target);
}
```

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L115-L127

## Recommended Mitigation Steps
Consider to change the comments that only `BLACKLIST_MANAGER_ROLE` by default.


# [NC-02] Unnecessary checks.

There is no need to check that `to != address(0)` since it already implemented in ERC20 `_mint()` function.

```solidity
function redistributeLockedAmount(address from, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && !hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {
        uint256 amountToDistribute = balanceOf(from);
        _burn(from, amountToDistribute);
        // to address of address(0) enables burning
        if (to != address(0)) _mint(to, amountToDistribute); // @audit address(0) check already implemented in `_mint()` function 

        emit LockedAmountRedistributed(from, to, amountToDistribute);
    } else {
      revert OperationNotAllowed();
    }
}
```

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L153


Same situation with `_beforeTokenTransfer`. No need to check `to != address(0)` since already implemented in ERC20 `_transfer` function.

```solidity
function _beforeTokenTransfer(address from, address to, uint256) internal virtual override {
    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && to != address(0)) { // @audit check already implemented in `transfer()` function
        revert OperationNotAllowed();
    }
    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {
        revert OperationNotAllowed();
    }
  }
```

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L245-L252


In `verifyRoute()` function there is unnecessary check that `route.addresses[i] == address(0)`, because it checked when `addCustodianAddress()` function is used.

```solidity
for (uint256 i = 0; i < route.addresses.length; ++i) {
    if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0) // @audit no need to check address(0)
    {
      return false;
    }
    totalRatio += route.ratios[i];
  }
```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L364

## Recommended Mitigation Steps
Consider to remove Unnecessary checks.


# [NC-03] Typos.

There is a typos in comment. 

```solidity
/// @notice Adds an custodian to the supported custodians list. // @audit an => a.
```
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L297

## Recommended Mitigation Steps
Consider to correct it to:

```solidity
/// @notice Adds a custodian to the supported custodians list. 
```
