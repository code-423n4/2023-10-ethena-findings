# Low

### StakedUSDe.sol: `redistributeLockedAmount()` to address(0) is burning the staked shares, and redistributing possible compromised funds to all users.

According to the sponsor comment on the main page, `FULL_RESTRCITED_STAKER_ROLE` is for sanction/stolen funds, or there is a request from law enforcement to freeze funds. We can see how this happens in the aforementioned function below.

```solidity
  function redistributeLockedAmount(address from, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && !hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {
      uint256 amountToDistribute = balanceOf(from);
      _burn(from, amountToDistribute);
      // to address of address(0) enables burning
      if (to != address(0)) _mint(to, amountToDistribute);

      emit LockedAmountRedistributed(from, to, amountToDistribute);
    } else {
      revert OperationNotAllowed();
    }
  }
```

If `to` is not set as address zero, then the restricted user shares will be burned and new ones will be minted to someone else. If `to` is set to address zero, only their shares are burned. If their shares are burned with no concerns to the underlying, the price per share will increase, resulting in treating the restricted user funds as part of the protocol's yield. As a result, users will be able to withdraw part of these compromised funds.

### EthenaMinting.sol: Assets which are removed from the contract will be stuck in it

In the minting contract, there's a function shown below which is used to remove supported assets for minting.

```solidity
  function removeSupportedAsset(address asset) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (!_supportedAssets.remove(asset)) revert InvalidAssetAddress();
    emit AssetRemoved(asset);
  }
```

However if some collateral deposited in the contract for redemeption purposes is removed, the assets will be stuck in it unless `DEFAULT_ADMIN_ROLE` reintroduces the asset and submits an order for redemption.

### StakedUSDeV2.sol: Initial cooldown setting differs from the documentation

According to the sponsor comment on the main contest page:

>There's also an additional change to add a 14 day cooldown period on unstaking stUSDe.

However as can be seen in the constructor code shown below:

```solidity
  constructor(IERC20 _asset, address initialRewarder, address owner) StakedUSDe(_asset, initialRewarder, owner) {
    silo = new USDeSilo(address(this), address(_asset));
    cooldownDuration = MAX_COOLDOWN_DURATION; 
  }
```
The cooldown is set as the maximum duration possible, which is 90 days. Considering the code has no initialization methods, this means the contract is live as soon as it's deployed, and the cooldown will be set to this erroneous value until the admin calls `setCooldownDuration()`, possibly affecting the first depositors.

# Non-Critical

### EthenaMinting.sol: `verifyNonce()` collides nonces every `2^64` entries.

The way the contract tracks unique nonces is by storing an invalidator amount and an invalidator bit as index. But there's an uint64 conversion to the nonce, which means values above `2^64^` will silently overflow.

So for example, if 2^64 + 1 is submitted, this will invalidate the nonce of number 1. According to the developers in the discord channel, a random number can be submitted as a nonce. This behaviour should be discouraged in favour of a sequential nonce submission in order to prevent a possible collision. Alternatively, remove the type conversion to uint64.
