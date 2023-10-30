* In the EthenaMinting.sol, there is no way to set a new minter and redeemer role address after the previous one has been removed by the GATEKEEPER in case the wallet private keys are compromised.

  function removeMinterRole(address minter) external onlyRole(GATEKEEPER_ROLE) {
    _revokeRole(MINTER_ROLE, minter);
  }

  function removeRedeemerRole(address redeemer) external onlyRole(GATEKEEPER_ROLE) {
    _revokeRole(REDEEMER_ROLE, redeemer);
  }


* In stakedUSDe.sol, the comment of the functions addToBlacklist() and removeFromBlacklist() states that owner and BLACKLIST_MANAGER can blacklist and un-blacklist addresses, but the onlyRole modifier only checks for if the caller is the BLACKLIST_MANAGER and the notOwner modifier checks for if the target is the Owner.

 /**
   * @notice Allows the owner (DEFAULT_ADMIN_ROLE) and blacklist managers to blacklist addresses.
   * @param target The address to blacklist.
   * @param isFullBlacklisting Soft or full blacklisting level.
   */
  function addToBlacklist(address target, bool isFullBlacklisting)
    external
    onlyRole(BLACKLIST_MANAGER_ROLE)
    notOwner(target)
  {
    bytes32 role = isFullBlacklisting ? FULL_RESTRICTED_STAKER_ROLE : SOFT_RESTRICTED_STAKER_ROLE;
    _grantRole(role, target);
  }

  /**
   * @notice Allows the owner (DEFAULT_ADMIN_ROLE) and blacklist managers to un-blacklist addresses.
   * @param target The address to un-blacklist.
   * @param isFullBlacklisting Soft or full blacklisting level.
   */
  function removeFromBlacklist(address target, bool isFullBlacklisting)
    external
    onlyRole(BLACKLIST_MANAGER_ROLE)
    notOwner(target)
  {
    bytes32 role = isFullBlacklisting ? FULL_RESTRICTED_STAKER_ROLE : SOFT_RESTRICTED_STAKER_ROLE;
    _revokeRole(role, target);
  }


* In EthenaMinting.sol, Admin can update the values of either setMaxMintPerBlock() or setMaxRedeemPerBlock() or both of them, when the intent was just to enable mintRedeem after it has been disabled to the previous value. So create a separate function to enable mint redeem after it has been disabled instead of passing values into setMaxMintPerBlock() or setMaxRedeemPerBlock().

 
