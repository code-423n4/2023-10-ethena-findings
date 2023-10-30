# [G-01] Remove onlyOwner modifier from renounceOwnership

[USDe.sol#L33-L35](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDe.sol#L33-L35) 

The function is supposed to always revert. Removing the modifier saves 1600 gas on deployment.

# [G-02] Use hardcoded value instead of retrieving it from calldata 
[EthenaMinting.sol#L171](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L171)

`order.order_type` is guaranteed to be `OrderType.MINT` because of the check at line 169.

```diff
    if (order.order_type != OrderType.MINT) revert InvalidOrder();
    verifyOrder(order, signature);
-   if (!verifyRoute(route, order.order_type)) revert InvalidRoute();
+   if (!verifyRoute(route, OrderType.MINT)) revert InvalidRoute();
```

```diff
  | contracts/EthenaMinting.sol:EthenaMinting contract |                 |       |        |        |         |
  |----------------------------------------------------|-----------------|-------|--------|--------|---------|
  | Deployment Cost                                    | Deployment Size |       |        |        |         |
- | 3576457                                            | 18793           |       |        |        |         |
+ | 3575057                                            | 18786           |       |        |        |         |
```
# [G-03] Remove adding zero
[StakedUSDe.sol#L90-L91](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L90-L91)

`getUnvestedAmount()` returns uint256. If it returns anything but zero, the execution reverts.

```diff
    if (getUnvestedAmount() > 0) revert StillVesting(); //  
-   uint256 newVestingAmount = amount + getUnvestedAmount();
+   uint256 newVestingAmount = amount;
```
# [G-04] Redundant casting to address
[StakedUSDe.sol#L138-L141](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L138-L141)
```diff
  function rescueTokens(address token, uint256 amount, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {
-   if (address(token) == asset()) revert InvalidToken();
+   if (token == asset()) revert InvalidToken();
```

# [G-05] EthenaMinting#verify - unnecessary downcasting

[EthenaMinting.sol#L379](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L379)

Downcasting to uint64 is absolutely not needed. It limits the space of nonces from `2^256 - 1` to `2^64 - 1` and uses slightly more gas.

```diff
- uint256 invalidatorSlot = uint64(nonce) >> 8;
+ uint256 invalidatorSlot = uint64(nonce) >> 8;
```
```diff
  | contracts/EthenaMinting.sol:EthenaMinting contract |                 |       |        |        |         |
  |----------------------------------------------------|-----------------|-------|--------|--------|---------|
  | Deployment Cost                                    | Deployment Size |       |        |        |         |
- | 3576457                                            | 18793           |       |        |        |         |
+ | 3574657                                            | 18784           |       |        |        |         |
```

# [G-06] Use uint256 instead of uint104 for UserCooldown.cooldownEnd

[IStakedUSDeCooldown.sol#L8](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/interfaces/IStakedUSDeCooldown.sol#L8)

```diff
struct UserCooldown {
-   uint104 cooldownEnd;
+   uint256 cooldownEnd;
    uint256 underlyingAmount;
}
```

Remove downcasting:

[StakedUSDeV2.sol#L100](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L100)

[StakedUSDeV2.sol#L116](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDeV2.sol#L116)

Change uint104 to uint256:

[StakedUSDeV2.blacklist.t.sol#L95](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/test/foundry/staking/StakedUSDeV2.blacklist.t.sol#L95)

[StakedUSDeV2.cooldownEnabled.t.sol#L88](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/test/foundry/staking/StakedUSDeV2.cooldownEnabled.t.sol#L88)

[StakedUSDeV2.cooldownEnabled.t.sol#L110](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/test/foundry/staking/StakedUSDeV2.cooldownEnabled.t.sol#L110)

```diff
  | contracts/StakedUSDeV2.sol:StakedUSDeV2 contract |                 |       |        |       |         |
  |--------------------------------------------------|-----------------|-------|--------|-------|---------|
  | Deployment Cost                                  | Deployment Size |       |        |       |         |
- | 3773992                                          | 20505           |       |        |       |         |
+ | 3725335                                          | 20262           |       |        |       |         |
```
# [G-07] Dead code

`EthenaMinting#_deduplicateOrder` either reverts or returns true. The `!_deduplicateOrder` check is not needed.

[EthenaMinting.sol#L172](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L172)
[EthenaMinting.sol#L203](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L203)
```diff
-   if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();
+   _deduplicateOrder(order.benefactor, order.nonce));
```
[IEthenaMinting.sol#L40](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/interfaces/IEthenaMinting.sol#L40)
```diff
-   error Duplicate();
```
[MintingBaseSetup.sol#L78](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/test/foundry/minting/MintingBaseSetup.sol#L78)
```diff
-  bytes internal Duplicate = abi.encodeWithSelector(IEthenaMinting.Duplicate.selector);
```
```diff
  | contracts/EthenaMinting.sol:EthenaMinting contract |                 |       |        |        |         |
  |----------------------------------------------------|-----------------|-------|--------|--------|---------|
  | Deployment Cost                                    | Deployment Size |       |        |        |         |
- | 3576457                                            | 18793           |       |        |        |         |
+ | 3555233                                            | 18687           |       |        |        |         |
```
