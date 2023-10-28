## Tittle: Role management lacks events, hindering transparency and accountability in access.

## Impact
Role management functions don't emit events for clarity.

The contract has several functions for managing access control roles: 

```solidity
function grantRole(bytes32 role, address account) 
function revokeRole(bytes32 role, address account)
```

However, these do not emit any events when roles are granted or revoked:

```solidity
// No events emitted!
function revokeRole(bytes32 role, address account) {
  // ...
  _revokeRole(role, account)
}
```

This reduces transparency into role changes over time:

- No historical log of who granted/revoked roles
- Cannot detect suspicious role changes  
- Reduces accountability around role management

## Proof of Concept

**Lack of Events for Role Changes in StakedUSDe**

The main role management functions are: https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/SingleAdminAccessControl.sol#L41-L43

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/SingleAdminAccessControl.sol#L50-L52

```solidity
// Grant role
function grantRole(bytes32 role, address account) external {
  _grantRole(role, account);
}

// Revoke role  
function revokeRole(bytes32 role, address account) external {
  _revokeRole(role, account);
}
```

Neither function emits any events:

```solidity
function grantRole(bytes32 role, address account) external {

  // No event emitted!
  _grantRole(role, account); 

}
```

This means there is no on-chain log when roles change:

```
1. Admin grants REWARDER role to Alice
2. No event emitted
3. Hard to detect role change after the fact
```

## Recommended Mitigation Steps

To improve, emit events on role changes:

```solidity
// Emit event on change
event RoleGranted(bytes32 role, address account, address changedBy);

function grantRole(bytes32 role, address account) external {

  emit RoleGranted(role, account, msg.sender);

  _grantRole(role, account);
} 
```

Now there is an immutable record of all role changes:

```
1. Admin grants REWARDER role to Alice
2. RoleGranted event emitted 
3. Can detect change in blockchain logs
```

Emitting events creates transparency into role changes over time. This provides accountability and oversight for administrative functions.

---

## Title: Blacklisting restricts actions but doesn't affect existing shares or benefits.

## Impact

Blacklisting only prevents new **stakes/unstakes/transfers**, but doesn't slash or burn existing shares.

## Proof of Concept

The blacklisting only limits future actions but doesn't affect existing shares. The blacklist functionality revokes a user's ability to call certain functions: [_revoke](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L126), 

```solidity
// Revoke staking role
    _revokeRole(role, target);

// This will revert if user tries to stake
function stake() onlyRole(STAKER_ROLE) {
  // ...
}
```

However, it does not burn or slash any existing shares the user has already:

```solidity
// No burning of shares
function revokeStaking(address user) {
  revokeRole(STAKER_ROLE, user);
  
  // User keeps all existing shares
  // balanceOf(user) remains the same
}
```

This means blacklisted users keep all prior benefits:

- They keep staked tokens and can wait out blacklist
- They keep earning yield on existing shares
- They can still vote in governance


**Blacklisting in StakedUSDe**

Blacklisting revokes a user's STAKER_ROLE:  

```solidity
// Revoke staking role
function blacklist(address user) external {

  revokeRole(STAKER_ROLE, user);

}
```

This prevents them calling `stake()` and `unstake()`:

```solidity
// Reverted for blacklisted user
function stake() onlyRole(STAKER_ROLE) {
  // ...
}

function unstake() onlyRole(STAKER_ROLE) {
  // ...
}
```

However, it does NOT affect their existing shares:

```solidity
// User keeps all shares
function blacklist(address user) external {

  revokeRole(STAKER_ROLE, user);

  // No change to shares owned by user
  // user.balanceOf() remains the same

}
```

The user keeps earning yield on these shares:

```solidity
// Blacklisted user keeps earning yield 
function distributeYield() external {

  for (address user : allUsers) {
    // Blacklisted users still earn yield
    distribute(user); 
  }

}
```

And can still vote in governance:

```solidity
// Blacklisted users can still vote
function vote(uint proposalId) onlyHolders { 
  // proposal logic
}
```

## Recommended Mitigation Steps

Some improvements:

- Burn a % of shares when blacklisted 
- Prevent blacklisted users from voting
- Disable yield for blacklisted users

Only restricting future actions is not enough. Their existing shares and power should be limited too.

Blacklisting should burn/freeze existing stake, not just limit future actions. Otherwise it has limited impact on bad actors who already have large stakes.
