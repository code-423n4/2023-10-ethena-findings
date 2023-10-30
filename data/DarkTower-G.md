## Summary
### Gas Optimizations
|Tag|Issue|Contexts|
|-|:-|:-|
|G-1|Unnecessary `orderType` check in the `verifyRoute` function|1|

## [G-1]  Unnecessary `orderType` check in the `verifyRoute` function
`orderType` check in the `verifyRoute` function can be removed. The lines of code 353-355 returns true if the orderType param for the `verifyRoute` function is a `REDEEM` type. However, we only call this function to verify the route when minting and during this action, the orderType must be `MINT` as we have enforced a check for this case prior in the `mint` function at line 169. 

```solidity
  function mint(Order calldata order, Route calldata route, Signature calldata signature) {
    // @>  Line 169: 
    if (order.order_type != OrderType.MINT) revert InvalidOrder();
  } 
```

So, when we make it past the above check, we essentially no longer need to return true if the `orderType` is `REDEEM` in the lines of code below and thus the if statement is basically wasting gas:

```solidity
  function verifyRoute(Route calldata route, OrderType orderType) {
    // @> Line 354: 
    if (orderType == OrderType.REDEEM) {
      return true;
    }
  }
```