### [G-01] unnecessary check in the `verifyRoute()` function can be removed to save gas 
the function `verifyRoute` only get called by the function `mint()` which reverts in case of the orderType is Redeem , so the internal check if the orderType is redeem can be removed 
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L353-L355
```solidity 
    if (orderType == OrderType.REDEEM) {
      return true;
    }
```
### [G-02] perform the checks first will save gas in case of revert . 
in the function `verifyOrder()` the require statements should get executed first 
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L342-L347
```solidity 
  function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
    bytes32 taker_order_hash = hashOrder(order);
    address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
    if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
    if (order.beneficiary == address(0)) revert InvalidAmount();
    if (order.collateral_amount == 0) revert InvalidAmount();
    if (order.usde_amount == 0) revert InvalidAmount();
    if (block.timestamp > order.expiry) revert SignatureExpired();
    return (true, taker_order_hash);
  }
```
### [G-03] unnecessary function calls can be removed to save gas 
in the function `transferInRewards()` the function checks and reverts in case that the `getUnvestedAmount` is greater than zero so the function will only pass if the value of the function `getUnvestedAmount()` is equal to zero , but when the function `transferInRewards()` calculates the `newVestingAmount` the function adds the value of `getUnvestedAmount` which will always be equal to zero 
so this call to the `getUnvestedAmount` when calculating the `newVestingAmount` can be removed to save gas . 
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L89-L99
```solidity 
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
@>  if (getUnvestedAmount() > 0) revert StillVesting();
@>  uint256 newVestingAmount = amount + getUnvestedAmount();


    vestingAmount = newVestingAmount;
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);


    emit RewardsReceived(amount, newVestingAmount);
  }
``` 
### [G-04] unnecessary modifier can be removed in order to save gas 
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L123 

the function `removeFromBlacklist()` revoke the blacklist role from the `target` address , so due to preventing the owner from being blacklisted , the modifier `notOwner` that checks of that the target is not the owner can be removed . 
```solidity
  function removeFromBlacklist(address target, bool isFullBlacklisting)
    external
    onlyRole(BLACKLIST_MANAGER_ROLE)
    notOwner(target)
  {
    bytes32 role = isFullBlacklisting ? FULL_RESTRICTED_STAKER_ROLE : SOFT_RESTRICTED_STAKER_ROLE;
    _revokeRole(role, target);
  }
```

  