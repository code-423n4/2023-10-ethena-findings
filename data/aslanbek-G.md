# [G-01] Remove onlyOwner modifier from renounceOwnership

[USDe.sol#L33-L35](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDe.sol#L33-L35) 

The function reverts anyways. Removing the modifier saves 1600 gas on deployment.

# [G-02] Use hardcoded value instead of retrieving it from calldata 
[EthenaMinting.sol#L171](https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/EthenaMinting.sol#L171)

`order.order_type` is guaranteed to be `OrderType.MINT` thanks to the check at line 169.

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
  | Function Name                                      | min             | avg   | median | max    | # calls |
- | mint                                               | 4657            | 80639 | 123779 | 195520 | 38      |
+ | mint                                               | 4657            | 78735 | 123632 | 195373 | 39      |
```
