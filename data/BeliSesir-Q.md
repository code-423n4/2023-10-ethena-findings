## Reverting with incorrect error in EthenaMinting.sol


On line 345 of `EthenaMinting.sol` when checking if the `order.beneficiary` is the zero address, if the check fails the function is reverting with `InvalidAmount()` although it appears that it should be `InvalidZeroAddress()`.

## Recommend checking the soft and hard black list roles in the verifyOrder function

Although the overview states: 

> By design, Ethena will be the only ones calling mint(),redeen() and other functions in this contract.

I recommend checking the `order.beneficiary` does have the `SOFT_RESTRICTED_STAKER_ROLE` or `FULL_RESTRICTED_STAKER_ROLE` as an added layer of protection against accidentally minting tokens for a user in a restricted country or redeeming them for a frozen account.

## Consider emitting the AdminTransferred event after the admin has been successfully changed.

In `SingleAdminAccessControl.sol` file line 72 `_grantRole` function, consider emitting the AdminTransferred event last to prevent an erroneous log from being broadcasted in case the `_revokeRole` function were to fail. The modification can be done as follows:

    function _grantRole(bytes32 role, address account) internal override {
        if (role == DEFAULT_ADMIN_ROLE) {
            _revokeRole(DEFAULT_ADMIN_ROLE, _currentDefaultAdmin);
            _currentDefaultAdmin = account;
            delete _pendingDefaultAdmin;
            emit AdminTransferred(_currentDefaultAdmin, account);
        }
        super._grantRole(role, account);
    }

## Recommend allowing the owner to be un-blacklisted

In `StakedUSDe.sol`, in case an attacker were to somehow black list the owner, recommend allowing the black list manager to un-blacklist the owner as there does not seem to be any advantage to preventing the black list manager from doing so. This can be done by removing the `notOwner` modifier as follows:

    function removeFromBlacklist(address target, bool isFullBlacklisting)
        external
        onlyRole(BLACKLIST_MANAGER_ROLE)
    {
        bytes32 role = isFullBlacklisting ? FULL_RESTRICTED_STAKER_ROLE : SOFT_RESTRICTED_STAKER_ROLE;
        _revokeRole(role, target);
    }

## Unnecessary operation on line 91 of StakedUSDe.sol

In the `transferInRewards` function, the value of the call to `getUnvestedAmount()` appears guaranteed to be zero, based on the `if (getUnvestedAmount() > 0) revert StillVesting();` statement on line 90. So adding the value to the `newVestingAmount` on line 91, `uint256 newVestingAmount = amount + getUnvestedAmount();` seems to be useless. If the expectation is that the value of `getUnvestedAmount()` can be greater than zero, then the condition on line 90 should be reevaluated and changed to allow non-zero values without reverting. 