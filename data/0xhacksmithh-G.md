### [Gas-0] `keccak256` result should be `immutable` instead of `constant`
```diff
```
```
```

### [Gas-0] State Variable could be tightly packed
```diff
```
```
```

### [Gas-0] Instead of setting 0(default value) we can use `delete` keyword which is more gas efficient
```diff
```
```
```

### [Gas-0] Before making changes to state variable, you should run pre-existing check(updated value != current value)
```diff
```
```
```

### [Gas-0] No need to go for check `if (orderType == OrderType.REDEEM) {` in `verifyRoute()` as this function only called by `mint()` and orderType check already happen in that function prior to `verifyRoute()`
```diff
```
```
```

### [Gas-0] Some `memory` variable initialized with default value
```diff
```
```
```

### [Gas-0] Those Mathematical operation which will not overflow could mark as `unchecked`
```diff
```
```
```

### [Gas-0] Instead of `> 0` use `!= 0`, which is more gas efficient
```diff
```
```
```


### [Gas-0] `emits` could be more optimizable
```diff
```
```
```

### [Gas-0] While calculating `newVestingAmount` in `transferInRewards()` no need to add `getUnvestedAmount()` with inputed amount as `getUnvestedAmount() = 0` in that case
```diff
```
```
```

### [Gas-0] `MAX_COOLDOWN_DURATION` could be a `constant` state variable
```diff
```
```
```

### [Gas-0] Some state variable could be `private` to save some gas
```diff
```
```
```
