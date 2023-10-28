Title: Blacklist roles need robust access control, reversible restrictions, and appeals.

## Impact

There are two restriction roles:

- SOFT_RESTRICTED_STAKER_ROLE - Prevents depositing and withdrawing

- FULL_RESTRICTED_STAKER_ROLE - Prevents transferring, burning, or redeeming shares

The roles are managed via: [addToBlacklist](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L106-L113) & [removeFromBlacklist](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L120-L127)

```solidity
function addToBlacklist(address target, bool isFullBlacklisting) external {
  // Grant soft or full restriction role  
}

function removeFromBlacklist(address target, bool isFullBlack) external {
  // Revoke soft or full restriction role
}
```

Only `BLACKLIST_MANAGER_ROLE` can add/remove restrictions.

**The Concerns are**

- Need proper access control around BLACKLIST_MANAGER_ROLE

- Could restrict legitimate users by mistake 

- Need a process to appeal restrictions

- Restrictions are irreversible without manager

## Proof of Concept

When the BLACKLIST_MANAGER_ROLE is granted.

```solidity
// StakedUSDe constructor
```

If the address is a single key, they have full control over restrictions.

This could allow:

- Accidentally restricting legitimate users
- Maliciously restricting users for profit 
- Collusion with users to profit from restrictions

Better practice is to use a multi-sig wallet for BLACKLIST_MANAGER_ROLE:

```solidity 
// Multi-sig wallet
address public constant BLACKLIST_MANAGER = 0x123... 

constructor() {
  _grantRole(BLACKLIST_MANAGER_ROLE, BLACKLIST_MANAGER);
}
```

Requiring 2/3 signatures from independent entities prevents a single point of control.

**Irreversible** 

The current restriction roles are irreversible without the manager:

```solidity
// Only manager can unrestrict
function removeFromBlacklist(address target) onlyRole(BLACKLIST_MANAGER) external {
  // Revoke restriction role
}
```

This prevents even the owner from removing improper restrictions.

An option is to allow the owner to force unrestriction:

```solidity
// Owner can override manager
function forceUnrestrict(address target) onlyOwner external {
   _revokeRole(RESTRICTED_ROLE, target); 
}
```

## Recommended Mitigation Steps

- BLACKLIST_MANAGER_ROLE should be a multi-sig

- Explicit guidelines for applying restrictions

- Method for appealing restrictions

- Owner can overwrite restrictions as a fallback