# [QA-01] Lack of checking for address(0) when setting new minter

[File: USDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDe.sol#L23)
```
  function setMinter(address newMinter) external onlyOwner {
    emit MinterUpdated(newMinter, minter);
    minter = newMinter;
  }
```

Function `setMinter()` does not check if `minter` is `address(0)`.


# [QA-02] Lack of checking for address(0) when creating USDeSilo

[File: USDeSilo.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDeSilo.sol#L18)
```
  constructor(address stakingVault, address usde) {
    STAKING_VAULT = stakingVault;
    USDE = IERC20(usde);
  }
```

`constructor()` does not verify if `stakingVault` and `usde` are not address(0).

# [QA-03] Protocol does not allow to add additional users with `DEFAULT_ADMIN_ROLE` role

This issue is related to centralization risk (it should be also noted in the Analysis Report).
Protocol does not allow to create additional admins.

[File: SingleAdminAccessControl.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/SingleAdminAccessControl.sol#L41)
```
  function grantRole(bytes32 role, address account) public override onlyRole(DEFAULT_ADMIN_ROLE) notAdmin(role) {
    _grantRole(role, account);
  }

```
It's not possible to call `grantRole()` to set another admin, since `grantRole()` function implements `notAdmin(role)` modifier. This implies, that we cannot grant anyone with `DEFAULT_ADMIN_ROLE`.



# [QA-04] `addToBlacklist()` should be better defined

In the current implementation - function `addToBlacklist()` set's `target` role to be either `FULL_RESTRICTED_STAKER_ROLE` or `SOFT_RESTRICTED_STAKER_ROLE`.
Those roles are defined in the README.md:

```
Due to legal requirements, there's a SOFT_RESTRICTED_STAKER_ROLE and FULL_RESTRICTED_STAKER_ROLE. The former is for addresses based in countries we are not allowed to provide yield to, for example USA. 
Addresses under this category will be soft restricted. They cannot deposit USDe to get stUSDe or withdraw stUSDe for USDe. However they can participate in earning yield by buying and selling stUSDe on the open market.

FULL_RESTRCITED_STAKER_ROLE is for sanction/stolen funds, or if we get a request from law enforcement to freeze funds. Addresses fully restricted cannot move their funds, and only Ethena can unfreeze the address. 
```

Another function - `removeFromBlacklist()` - allows to revoke those, previously set roles.

By definition - `addToBlacklist()` and `removeFromBlacklist()` can be called by Blacklist Manager (modifier: `onlyRole(BLACKLIST_MANAGER_ROLE)`).

There is some inaccuracy in definition and description of these functions. According to the description - when `addToBlacklist()` is called on user - he should either be soft or full restricted. However, the current scenario does not take into consideration what will happen, when that user is one of the Blacklist Manager.

If Blacklist Manager was blacklisted by `FULL_RESTRICTED_STAKER_ROLE` role - he - by the definition of `addToBlacklist()`, should not be able to move his funds.
However, since he's still Blacklist Manager, he can remove himself from the blacklist by calling `removeFromBlacklist()` on his/her own address. That way he/she won't be on Blacklist anymore.


There are multiple of ways to solve this issue.
Whenever blacklisting active Blacklist Manager - either revoke his/her `BLACKLIST_MANAGER_ROLE` role - or, state in the documentation, that blacklisting active Blacklist Manager should be preceded by revoking that role.

Another idea would be adding additional check in `removeFromBlacklist()`: `require(msg.sender != target, "Cannot remove themself")`, to make sure, that user cannot remove himself/herself from the Blacklist.


# [QA-5] Incorrect revert  in `verifyOrder()` in `EthenaMinting.sol`

[File: EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L343)
```
    if (order.beneficiary == address(0)) revert InvalidAmount();
```

When `order.beneficiary == address(0)`, function `verifyOrder()` reverts with `InvalidAmount()`.
This revert is reserved for incorrect amounts. It should be used either `InvalidAddress()` or `InvalidZeroAddress()` instead.


# [QA-6] User who's being added to the blacklist can front-run this operation

The main purpose of `addToBlacklist()` call is to add the user to the blacklist. When user is blacklisted (with `FULL_RESTRICTED_STAKED_ROLE`) - he basically cannot do anything. Address with this role can't burn his `stUSDe` tokens to unstake his `USDe` tokens, neither to transfer `stUSDe` tokens. His balance can be manipulated by the admin of `StakedUSDe`.
However, when users notices in the mem-pool that `addToBlacklist()` is called - he can front-run it.

```
1. BLACKLIST_MANAGER_ROLE calls addToBlacklist(0xAAA, true)
2. User 0xAAA sees that transaction in the mem-pool
3. User 0xAAA front-runs that call and transfers stUSDe tokens to different address
4. Now addToBlacklist(0xAAA, true) is being called, 0xAAA is FULL_RESTRICTED_STAKER_ROLE, but he/she doesn't care, because he/she had already transferred their stUSDe tokens to different address
```

Ability to front-run `addToBlacklist()` allows to evade `FULL_RESTRICTED_STAKER_ROLE`.

# [QA-7] `SOFT_RESTRICTED_STAKER_ROLE` can by bypassed

According to documentation - this might be a design choice - however, since this choice influences the security of the protocol - I think it should be contained in the QA Report.

Blacklisted address (with `SOFT_RESTRICTED_STAKER_ROLE`) cannot deposit funds. However, nothing stops that address for transferring their funds to a different (non-blacklisted) address and deposit from that new address. This makes this role easily to be bypassed.

According to docs, the purpose of this role seems to be just for some legal requirements/regulations:

```
Due to legal requirements, there's a SOFT_RESTRICTED_STAKER_ROLE and FULL_RESTRICTED_STAKER_ROLE. The former is for addresses based in countries we are not allowed to provide yield to, for example USA. Addresses under this category will be soft restricted.
```

However, I think that it might be a good idea, to make this role a little more restrictive.

# [QA-8] Before adding/removing role, make sure that user already has that role

Whenever we `_grantRole` via `addToBlacklist()` function - there's no check if user already has that role. This implies, that it's possible to assign the same role to the user who already has that role.

Whenever we `_revokeRole` via `removeFromBlacklist()` function - there's no check if user really has that role. This implies, that it's possible to revoke a role from user who hasn't got that role (in that case, nothing will change, if user doesn't have a role and `removeFromBlacklist()` is being called on him - he still won't have that role).

[File: StakedUSDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L106)
```
 function addToBlacklist(address target, bool isFullBlacklisting)
    external
    onlyRole(BLACKLIST_MANAGER_ROLE)
    notOwner(target)
  {
    bytes32 role = isFullBlacklisting ? FULL_RESTRICTED_STAKER_ROLE : SOFT_RESTRICTED_STAKER_ROLE;
    _grantRole(role, target);
  }

 (...)

  function removeFromBlacklist(address target, bool isFullBlacklisting)
    external
    onlyRole(BLACKLIST_MANAGER_ROLE)
    notOwner(target)
  {
    bytes32 role = isFullBlacklisting ? FULL_RESTRICTED_STAKER_ROLE : SOFT_RESTRICTED_STAKER_ROLE;
    _revokeRole(role, target);
  }
```

Whenever we grant role to `target` (by calling `addToBlacklist()`), we should check if `target` already hasn't that role.
Whenever we revoke role from `target` (by calling `removeFromBlacklist()`), we should check if `target` already has that role.


# [QA-9] Lack of event emission when adding to/removing from the blacklist

In file `StakedUSDe.sol`, functions `addToBlacklist()` and `removeFromBlacklist()` do not emit any events.
Consider emitting events whenever user is being blacklisted/removed from the blacklist

# [N-01] Typo in documentation

[File: README.md](https://github.com/code-423n4/2023-10-ethena/blob/main/README.md)
```
The primary functions used in this contract is `mint()` and `redeen()`. Users who call this contract are all within Ethena. 
When outside users wishes to mint or redeem, they perform an EIP712 signature based on an offchain price we provided to them. 
They sign the order and sends it back to Ethena's backend, where we run a series of checks and are the ones who take their signed order and put them on chain.

By design, Ethena will be the only ones calling `mint()`,`redeen()` and other functions in this contract.
```

Function `redeen()` should be renamed to `redeem()`. There's no `redeen()` function in the code base.


# [N-02] `SingleAdminAccessControl.sol` is not fully compliant with `ERC-5313`

[File: SingleAdminAccessControl.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/SingleAdminAccessControl.sol#L62)
```
  /**
   * @dev See {IERC5313-owner}.
   */
  function owner() public view virtual returns (address) {
    return _currentDefaultAdmin;
  }
```

According to code-section - `owner()` is required for ERC-5313.
However, when we look at ERC-5313, it defines `owner()` as `external` - not `public`:

```
source: https://eips.ethereum.org/EIPS/eip-5313

Every contract compliant with this EIP MUST implement the EIP5313 interface.

// SPDX-License-Identifier: CC0-1.0
pragma solidity ^0.8.15;

/// @title EIP-5313 Light Contract Ownership Standard
interface EIP5313 {
    /// @notice Get the address of the owner    
    /// @return The address of the owner
    function owner() view external returns(address);
}

```

`external` (not `public`) is also in the Open Zeppelin's interface (which `SingleAdminAccessControl.sol` imports):

```
interface IERC5313 {
    /**
     * @dev Gets the address of the owner.
     */
    function owner() external view returns (address);
}
``` 

Recommendation: change `public` to `external`.


# [N-03] Stick with one way of using curly-brackets for single-instruction `if` conditions

[File: StackedUSDeV2](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L112)
```
if (shares > maxRedeem(owner)) revert ExcessiveRedeemAmount();
```

[File: StackedUSDeV2](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L127-L129)
```
 if (duration > MAX_COOLDOWN_DURATION) {
      revert InvalidCooldown();
    }
```

When `if` condition wants to execute a single instruction, that instruction does not need to be inside `{}`.
E.g., those two syntax are valid:

```
if (CONDITION) INSTRUCTION;

if (CONDITION) {
  INSTRUCTION;
}
```

It's a good coding practice to stick to one way of `if`-syntax. In the above scenario, in `StakedUSDeV2.sol` - the first syntax is being used, however, line `127-129` use the second one.


# [N-04] Don't use short forms/contractions when naming events 

[File: USDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDe.sol#L34)
```
revert CantRenounceOwnership();
```

Renaming event from `CantRenounceOwnership()` to `CannotRenounceOwnership()` will be much more readable.


[File: StakedUSDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L57)
```
 if (target == owner()) revert CantBlacklistOwner();
```

Renaming event from `CantBlacklistOwner()` to `CannotBlacklistOwner()` will be much more readable.


# [N-05] Documentation does not properly describe available roles

[File: README.md](https://raw.githubusercontent.com/code-423n4/2023-10-ethena/main/README.md)
```
### Trusted Roles

- `USDe` minter - can mint any amount of `USDe` tokens to any address. Expected to be the `EthenaMinting` contract
- `USDe` owner - can set token `minter` and transfer ownership to another address
- `USDe` token holder - can not just transfer tokens but burn them and sign permits for others to spend their balance
- `StakedUSDe` admin - can rescue tokens from the contract and also to redistribute a fully restricted staker's `stUSDe` balance, as well as give roles to other addresses (for example the `FULL_RESTRICTED_STAKER_ROLE` role)
```

The `Trusted Roles` paragraph lists all available roles in the protocol. One of the mentioned role is:

```
`USDe` token holder - can not just transfer tokens but burn them and sign permits for others to spend their balance
```

However, token holder cannot be classified as "Trusted Role", since it can be anyone. The paragraph name suggests that "`USDe` token holder" is a Trusted Role.
Consider changing the name of the paragraph to "Available Roles", since it lists not only Trusted Roles, but every roles implemented by the protocol.


# [N-06] Protocol supports limited number of nonces

There's some deviation from how nonces are being stored and how nonces are being represented.

[File: EthenaMinting](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L78)
```
 mapping(address => mapping(uint256 => uint256)) private _orderBitmaps;
```

`_orderBitmaps` is a `uint256 => uint256` mapping, in `verifyNonce()`.


[File: EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L379)
```
uint256 invalidatorSlot = uint64(nonce) >> 8;
```

The max value of `invalidatorSlot` is (2**64 - 1 ) >> 8 = 72057594037927935.

There are much more available invalidator slots in `_orderBitmaps`, than the number of invalidator slots which can be derived from `nonce`. 
This may lead to some collision, e.g.: (1 and 18446744073709551617)