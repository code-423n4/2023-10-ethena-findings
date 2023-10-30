# QA Report

## Inconsistent functionality between `verifyOrder` and `verifyRoute`

In `EthenaMinting`, `verifyOrder` and `verifyRoute` are used to verify the `order` and `route` arguments passed to `mint`/`redeem` by the `REDEEMER`. While `verifyOrder` reverts if the order is invalid, `verifyRoute` returns false without reverting if the route is invalid. While this inconsistency does not pose any immediate risks in the current implementation of the minting contract, it creates confusion for devs/auditors and may lead to unexpected outcomes in future versions of the contract.

```solidity
File: contracts\EthenaMinting.sol

338:   /// @notice assert validity of signed order
339:   function verifyOrder(Order calldata order, Signature calldata signature) public view override returns (bool, bytes32) {
340:     bytes32 taker_order_hash = hashOrder(order);
341:     address signer = ECDSA.recover(taker_order_hash, signature.signature_bytes);
342:     if (!(signer == order.benefactor || delegatedSigner[signer][order.benefactor])) revert InvalidSignature();
343:     if (order.beneficiary == address(0)) revert InvalidAmount();
344:     if (order.collateral_amount == 0) revert InvalidAmount();
345:     if (order.usde_amount == 0) revert InvalidAmount();
346:     if (block.timestamp > order.expiry) revert SignatureExpired();
347:     return (true, taker_order_hash);
348:   }
349: 
350:   /// @notice assert validity of route object per type
351:   function verifyRoute(Route calldata route, OrderType orderType) public view override returns (bool) {
352:     // routes only used to mint
353:     if (orderType == OrderType.REDEEM) {
354:       return true;
355:     }
356:     uint256 totalRatio = 0;
357:     if (route.addresses.length != route.ratios.length) {
358:       return false;
359:     }
360:     if (route.addresses.length == 0) {
361:       return false;
362:     }
363:     for (uint256 i = 0; i < route.addresses.length; ++i) {
364:       if (!_custodianAddresses.contains(route.addresses[i]) || route.addresses[i] == address(0) || route.ratios[i] == 0)
365:       {
366:         return false;
367:       }
368:       totalRatio += route.ratios[i];
369:     }
370:     if (totalRatio != 10_000) {
371:       return false;
372:     }
373:     return true;
374:   }
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L338-L374

## Setting `StakedUSDeV2`'s `cooldownDuration` variable from non-zero to zero should remove any existing cooldowns

If the admin calls `StakedUSDeV2#setCooldownDuration` to set `cooldownDuration` to 0, users that have recently called either `cooldownAssets` or `cooldownShares` will still have to wait for the remainder of the previous cooldown duration to access their assets (USDe). This provides a poor user experience and is likely unintended.

A simple fix is to alter `unstake` to allow withdrawals from the silo whenever `cooldownDuration` is equal to 0.

```diff
  function unstake(address receiver) external {
    UserCooldown storage userCooldown = cooldowns[msg.sender];
    uint256 assets = userCooldown.underlyingAmount;

-   if (block.timestamp >= userCooldown.cooldownEnd) {
+   if (block.timestamp >= userCooldown.cooldownEnd || cooldownDuration == 0) {
      userCooldown.cooldownEnd = 0;
      userCooldown.underlyingAmount = 0;

      silo.withdraw(receiver, assets);
    } else {
      revert InvalidCooldown();
    }
  }
```
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L78-L90

## `StakedUSDe`'s `SOFT_RESTRICTED_STAKER_ROLE` can be trivially bypassed, rendering it redundant

If a user is added to the blacklist by being assigned the `SOFT_RESTRICTED_STAKER_ROLE`, they are unable to stake but still able to redeem/withdraw. This restriction can be bypassed by the user by simply redeeming/withdrawing their assets to a different address that they control, and staking from that address instead. Consider whether this behaviour is acceptable and rethink the implementation of restricted roles if not.

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L210-L212