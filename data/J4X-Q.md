# Low

## [L-01] Users can only be blacklisted or removed from blacklist by the blacklist manager

**Issue Description:**
The NatSpec documentation for the functions `addToBlacklist()` and `removeFromBlacklist()` inaccurately states, "Allows the owner (`DEFAULT_ADMIN_ROLE`) and blacklist managers to blacklist addresses." In practice, the `onlyRole(BLACKLIST_MANAGER_ROLE)` modifier permits only the holder of the `BLACKLIST_MANAGER_ROLE` to blacklist addresses, not the holder of the `DEFAULT_ADMIN_ROLE` role.

**Recommended Mitigation Steps:**
If the intended functionality aligns with the implemented code, the NatSpec description should be updated to reflect this and state that only blacklist managers can blacklist addresses.

If the intention is for both the holders of the `DEFAULT_ADMIN_ROLE` and `BLACKLIST_MANAGER_ROLE` to be able to call these functions, the require statement needs to be modified as follows:

```solidity
function addToBlacklist(address target, bool isFullBlacklisting)
external
notOwner(target)
{
	require(hasRole(BLACKLIST_MANAGER_ROLE, msg.sender) || hasRole(DEFAULT_ADMIN_ROLE, msg.sender))
	bytes32 role = isFullBlacklisting ? FULL_RESTRICTED_STAKER_ROLE : SOFT_RESTRICTED_STAKER_ROLE;
	_grantRole(role, target);
}
```

```solidity
function removeFromBlacklist(address target, bool isFullBlacklisting)
external
notOwner(target)
{
	require(hasRole(BLACKLIST_MANAGER_ROLE, msg.sender) || hasRole(DEFAULT_ADMIN_ROLE, msg.sender))
	bytes32 role = isFullBlacklisting ? FULL_RESTRICTED_STAKER_ROLE : SOFT_RESTRICTED_STAKER_ROLE;
	_revokeRole(role, target);
}
```

---


# Non-Critical

## [NC-01]  Comment missing dash
[EthenaMinting Line 68](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L68)

**Issue Description:**
The comment "// @notice custodian addresses" does not conform to the established style in the document, which uses three dashes before each `@notice`.

**Recommended Mitigation Steps:**
Revise the comment to adhere to the document's style by using three dashes before `@notice`, like this: "/// @notice custodian addresses."

---

## [NC-02]  Incorrect comment on `_orderBitmaps`
[EthenaMinting Line 78](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L78)

**Issue Description:**
The comment "/// @notice user deduplication" inaccurately describes the functionality of the variable `_orderBitmaps`. The variable is intended to track nonces for each user, not user deduplication.

**Recommended Mitigation Steps:**
Update the comment to accurately describe the functionality of the variable, as follows: "/// @notice user -> nonce deduplication."
