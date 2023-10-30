# [L] The return value of boolean is not needed

As seen from the function implementation, there is not possible to return a false boolean, as mostly they will just revert except the last line returning `true`.

Furthermore, if we see from the codebase, usage of `verifyOrder` function call, the return value of both bool and bytes32 is not being used anywhere. Maybe the bytes32 orderhash needed on off-chain mechanism, but the boolean is not necessary since there are two possible outcome, it will just revert or return true, thus on off-chain, if it's reverted, it's false, else true, so no need for this boolean.

```js
File: EthenaMinting.sol
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
```

# [L] Wrong revert name

Clearly as seen from the line below, the check is to make sure it's not a zero address, which is not align with the revert type an `InvalidAmount`

```js
File: EthenaMinting.sol
343:     if (order.beneficiary == address(0)) revert InvalidAmount();
```

# [L] Lack of checking duplicate routes

Eventhough this will eventually be called only by MINTER_ROLE or REDEEMER_ROLE, but this `verifyRoute` contains several checks, the length adresses, check if address is not zero, check if address is in custodianAdresses, but it's missing check if the address of routes is duplicates.

```js
File: EthenaMinting.sol
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
