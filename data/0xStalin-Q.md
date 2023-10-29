## [L-01] EthenaMinting contract could not have enough native token to cover the request to transferToCustody
- [The transferToCustody()](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L247-L262) is capable of sending the native token to the custodians, the problem is that the function relies on the fact that the EthenaMinting contract has enough native tokens to cover the requested amounts of native token to be transferred, which could not be the case, and tx will be reverted.

> EthenaMinting contract
```solidity
  function transferToCustody(address wallet, address asset, uint256 amount) external nonReentrant onlyRole(MINTER_ROLE) {
    if (wallet == address(0) || !_custodianAddresses.contains(wallet)) revert InvalidAddress();
    //@audit-issue => If EthenaMinting contract doesn't have enough native token, the tx will revert!
    if (asset == NATIVE_TOKEN) {
      (bool success,) = wallet.call{value: amount}("");
      if (!success) revert TransferFailed();
    } else {
      IERC20(asset).safeTransfer(wallet, amount);
    }
    emit CustodyTransfer(wallet, asset, amount);
  }
```

**Fix:**
- The fix for this issue is to make the transferToCustody() function payable, and instead of relying upon that the contract has enough native tokens, opt to send the exact amount of native tokens that will be sent to the custodian within the same tx

## [L-02] Not setting a boundary to prevent the contract from running out of custodians that can receive the user collateral
- [When removing custodians from the EthenaMinting contract](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L270-L273) there is not a validation to prevent the contract from running out of approved custodians.
  - If the contract has 0 approved custodians, the minting process will be impacted because there won't be any addresses to route the collateral deposited by the users

> EthenaMinting contract
```solidity
  //@audit-issue => Doesn't enforce a minimum amount of approved custodians, it only removes the given custodian's address
  function removeCustodianAddress(address custodian) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (!_custodianAddresses.remove(custodian)) revert InvalidCustodianAddress();
    emit CustodianAddressRemoved(custodian);
  }
```

**Fix:**
- The fix for this issue is to define a minimum number of valid custodians that must be in the contract, it could be by using a constant or a variable that could be updated by the admin in case this value needs to be updated, but, just make sure that the minimum number of approved custodians is never 0

## [L-03] Not setting boundaries to limit valid limits of the new values when updating the maxMintPerBlock & maxRedeemPerBlock variables
- Currently, there are no boundary limits for the accepted range of values that the maxMintPerBlock and maxRedeemPreBlock variables can be assigned, it is recommended to always define a valid range of valid values for these types of variables that can be updated and can impact the functionality of other functions.
  - [_setMaxMintPerBlock()](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L436-L440)
  - [_setMaxRedeemPerBlock()](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L443-L447)

> EthenaMinting contract
```solidity
  //@audit-issue => Not setting boundaries to limit the upper and lower limits of the new values!
  function _setMaxMintPerBlock(uint256 _maxMintPerBlock) internal {
    uint256 oldMaxMintPerBlock = maxMintPerBlock;
    maxMintPerBlock = _maxMintPerBlock;
    emit MaxMintPerBlockChanged(oldMaxMintPerBlock, maxMintPerBlock);
  }

  //@audit-issue => Not setting boundaries to limit the upper and lower limits of the new values!
  /// @notice Sets the max redeemPerBlock limit
  function _setMaxRedeemPerBlock(uint256 _maxRedeemPerBlock) internal {
    uint256 oldMaxRedeemPerBlock = maxRedeemPerBlock;
    maxRedeemPerBlock = _maxRedeemPerBlock;
    emit MaxRedeemPerBlockChanged(oldMaxRedeemPerBlock, maxRedeemPerBlock);
  }
```

**Fix:**
- The fix for this issue is to define a valid range of values that can be assigned to these two variables, and before changing their values, always validate that the new values are within the valid range of acceptable values.

## [L-04] No need to call twice the same function to get the unvested amount of rewards
- When the rewarder transfers rewards to the Staked contract, the transferInRewards() function [calls twice the getUnvestedAmount() function](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L90-L93), but, it is not really necessary to call it twice, if the first if conditional is passed, it means that the unvested period is over, and the unvested amount is 0, so, the returned value from the getUnvestedAmount() will be 0, thus, there is no point in calling it again when computing the value of the `newVestingAmount` variable, as a matter of fact, this variable is also not required, since the value will be the same as the value of the inputted `amount`.

**Fix:**
- Remove the second call to the getUnvestedAmount() and also delete the `newVestingAmount`, it is not required.
```solidity
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    //@audit-info => If the unvested amount is > 0, it means the previous vesting period is still active!
    if (getUnvestedAmount() > 0) revert StillVesting();

    //@audit-info => If reaches here, the previous vestin period is over, thus, vested amount is 0!
    //@audit-info => The new vesting amount is just amount!

-    uint256 newVestingAmount = amount + getUnvestedAmount();

-    vestingAmount = newVestingAmount;
+    vestingAmount = amount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

-   emit RewardsReceived(amount, newVestingAmount);
+   emit RewardsReceived(amount);
  }
```

## [L-05] When a custodian is removed from the list of valid custodians, any assets that were transferred to them are not claimed back
- [When custodians are removed](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L270-L273), currently the EthenaMinting contract is not getting back all the assets that were transferred to them.
- The function only removes the custodian from the valid list of custodians, but it doesn't enforce a transfer of all the user's collateral that was transferred to them.
```solidity
  function removeCustodianAddress(address custodian) external onlyRole(DEFAULT_ADMIN_ROLE) {
    //@audit-issue => Not pulling the funds that were transfered to the custodian
    if (!_custodianAddresses.remove(custodian)) revert InvalidCustodianAddress();
    emit CustodianAddressRemoved(custodian);
  }
```

**Fix:**
- Pull the funds that were transferred to the custodian before removing them from the list of valid custodians, and once those funds have been pulled into the EthenaMinting contract, make sure to distribute them to the rest of the valid custodians.