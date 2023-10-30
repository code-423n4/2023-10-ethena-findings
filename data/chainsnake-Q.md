## L-01. In `EthenaMinting::mint()` function, the order duplication error will never occur.
In `mint()` function, `Duplicate()` error is supposed to occur if `_deduplicateOrder()` returns false.
```
  function mint(Order calldata order, Route calldata route, Signature calldata signature)
    external
    override
    nonReentrant
    onlyRole(MINTER_ROLE)
    belowMaxMintPerBlock(order.usde_amount)
  {
    ...  ...
    if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();
    ...  ...
  }
```

But in `verifyNonce()`, if the given nonce duplicates, `InvalidNonce()` error occurs instead of returning false, which means it won't reach the above revert.

So I would suggest modifying `_deduplicateOrder()` and `verifyNonce()` like below so that they can return nonce validation:
```
  function verifyNonce(address sender, uint256 nonce) public view override returns (bool, uint256, uint256, uint256) {
    if (nonce == 0) revert InvalidNonce();

    uint256 invalidatorSlot = uint64(nonce) >> 8;
    uint256 invalidatorBit = 1 << uint8(nonce);
    mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
    uint256 invalidator = invalidatorStorage[invalidatorSlot];
    bool invalidNonce = (invalidator & invalidatorBit != 0);

    return (!invalidNonce, invalidatorSlot, invalidator, invalidatorBit);
  }

  function _deduplicateOrder(address sender, uint256 nonce) private returns (bool) {
    (bool valid, uint256 invalidatorSlot, uint256 invalidator, uint256 invalidatorBit) = verifyNonce(sender, nonce);
    if (valid) { // Update invalidatorBit only if the nonce is valid
      mapping(uint256 => uint256) storage invalidatorStorage = _orderBitmaps[sender];
      invalidatorStorage[invalidatorSlot] = invalidator | invalidatorBit;
    }
    return valid;
  }
```

## L-02. Should check if `nonce` is less than uint64.max in `EthenaMinting::verifyNonce()` function.
Verifying nonce is done under the assumption that the nonce is less than `uint64.max`. If nonce is increased one by one from the user side, it is difficult to be more than `uint64.max`, but if it is a random value, a duplication error may occur for different two nonces if one of them is more than `uint64.max`.

So I suggest adding check if the nonce is less than the limit.
```
  function verifyNonce(address sender, uint256 nonce) public view override returns (bool, uint256, uint256, uint256) {
    if (nonce == 0) revert InvalidNonce();
    if (nonce >= uint64.max) revert InvalidNonce(); // --> Add here
    ... ...
  }
```

## L-03. Unnecessary calculation of `getUnvestedAmount()` can be removed in `StakedUSDe::transferInRewards()`.
In `transferInRewards()` function, transferring rewards to the vault is prohibited when there is a remaining unvested amount.
``` if (getUnvestedAmount() > 0) revert StillVesting(); ```

But in the next line, an unnecessary calculation is performed which is always 0.
``` uint256 newVestingAmount = amount + getUnvestedAmount(); ```

So we can update the function like below:
```
  function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {
    if (getUnvestedAmount() > 0) revert StillVesting();

    vestingAmount = amount; // --> Here, update this line
    lastDistributionTimestamp = block.timestamp;
    // transfer assets from rewarder to this contract
    IERC20(asset()).safeTransferFrom(msg.sender, address(this), amount);

    emit RewardsReceived(amount, newVestingAmount);
  }
```

## L-04. Stakers can be doubly restricted by two restriction roles
By standard AccessControl, a user can have multiple roles and this is same for `FULL_RESTRICTED_STAKER_ROLE` and `FULL_RESTRICTED_STAKER_ROLE`.

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L106-L113
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L120-L127

Even if a staker was released from one restriction role, it can be restricted by the other role.
So when adding or removing a restriction role, then either role should be considered in `addToBlacklist()` and `removeFromBlacklist()`.
