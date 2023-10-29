
# [L-1] `EthenaMinting.verifyRoutedoes` not verify that there are duplicate elements in route[]

In the for loop does not verify that there are duplicate elements in route[], so the caller can pass in multiple routes of the same value, and the variable totalRatio can be added repeatedly.

``` solidity

    uint256 totalRatio = 0;
    .....
    for (uint256 i = 0; i < route.addresses.length; ++i) {
      if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)
      {
        return false;
      }
@>    totalRatio += route.ratios[i];
    }
```

# [L-2] EthenaMinting._transferCollateral will have a division overflow

When the amount * ratios[i] < 10_000,`amountToTransfer` is 0, so that the token amount received in addresses[i](route) is 0.

Since `totalTransferred` is recorded in the function, the end will transfer the amount of tokens that have not been transferred to the last address in the `addresses`, so there is no loss of funds.

But it may cause another problem:
For example, a caller in mint wants to assign a small percentage of the collateral token to another route, the token is addressed in addresses[0],
However, because the number is too small, division overflow occurs, the token amount received in addresses[0] is 0, and the tokens are all transferred to addresses[1], which is inconsistent with the result expected by the caller.

``` solidity
  function _transferCollateral(
    uint256 amount,
    address asset,
    address benefactor,
    address[] calldata addresses,
    uint256[] calldata ratios
  ) internal {
    .....
    for (uint256 i = 0; i < addresses.length; ++i) {
@>    uint256 amountToTransfer = (amount * ratios[i]) / 10_000;
@>    token.safeTransferFrom(benefactor, addresses[i], amountToTransfer);
      totalTransferred += amountToTransfer;
    }
    uint256 remainingBalance = amount - totalTransferred;
    if (remainingBalance > 0) {
      token.safeTransferFrom(benefactor, addresses[addresses.length - 1], remainingBalance);
    }
  }
```

# [N-1] No events are added to the StakedUSDeV2 public method

There are no emit events in public methods such as `unstake` `cooldownAssets` in StakedUSDeV2, only `setCooldownDuration` has an event log.

There is no such problem with the EthenaMinting contract.

``` solidity
  function unstake(address receiver) external {
    UserCooldown storage userCooldown = cooldowns[msg.sender];
    uint256 assets = userCooldown.underlyingAmount;

    if (block.timestamp >= userCooldown.cooldownEnd) {
      userCooldown.cooldownEnd = 0;
      userCooldown.underlyingAmount = 0;
      silo.withdraw(receiver, assets);
    } else {
      revert InvalidCooldown();
    }
  }

```

# [N-2] The grantRole modifier is confusing

SingleAdminAccessControl.grantRole has 2 Modifiers, onlyRole and notAdmin,
onlyRole like: Must be ADMIN,
notAdmin like: Cannot be ADMIN,
It looks like the opposite, it's confusing,
In fact, `msg.sender` is restricted by onlyRole, while notAdmin restricts the permission of the passed parameter `role`. It is recommended to add a description in the comment.

``` solidity
  /// @notice grant a role
  /// @notice can only be executed by the current single admin
  /// @notice admin role cannot be granted externally
  /// @param role bytes32
  /// @param account address
  function grantRole(bytes32 role, address account) public override onlyRole(DEFAULT_ADMIN_ROLE) notAdmin(role) {
    _grantRole(role, account);
  }
```