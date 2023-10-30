
| |Issue|Instances|
|-|:-|:-:|
| [LOW-1](#LOW-1) | ownership transfer  in constructor is not implemented on Ownable2Step way| 1 |
| [LOW-2](#LOW-2) | transaction fails on one of the variant of `token` & `to`| 1 |
| [LOW-3](#LOW-3) | FULL_RESTRICTED_STAKER_ROLE can be frontran | 1 |

### <a name="LOW-1"></a>[LOW-1] `USDe::constructor` ownership transfer for first time in constructor is not implemented on Ownable2Step method

- Initial ownership transfer should be done in Ownable2Step method, but directly transferred in [constructor](https://github.com/code-423n4/2023-10-ethena/blob/d361ced0f57005a1b80c7d01673e17582847aaed/protocols/USDe/contracts/USDe.sol#L20)
- First call `transferOwnership(admin)` then from admin key, call `acceptOwnership()`
- Severity : Low
- Likelihood : Medium

## Recommended Mitigation Steps
```diff
-  _transferOwnership(admin);
+  transferOwnership(admin);
```

### <a name="LOW-2"></a>[LOW-2] `StakedUSDe.rescueTokens()` transaction fails if `token` == `stUSDE` && `to` == address with `FULL_RESTRICTED_STAKER_ROLE`
- Severity : low
- Likelihood : low

## Recommended Mitigation Steps
- So either improve your backend to do all the checks, signature replays etc., Or change the way `EthenaMinting.setDelegatedSigner(address)` is implemented in 2 step process like ownable2step.
- So the delegated EOA sould accept the delegation and then only can be a delegate.
- Second mitigation requires introducing new storage and a new function to accept delegation.
```diff
  function rescueTokens(address token, uint256 amount, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (address(token) == asset()) revert InvalidToken();
-    IERC20(token).safeTransfer(to, amount);
  }

  function rescueTokens(address token, uint256 amount, address to) external onlyRole(DEFAULT_ADMIN_ROLE) {
    if (address(token) == asset()) revert InvalidToken();
+   if (address(token) != address(this) && !hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {   
+     IERC20(token).safeTransfer(to, amount);
+   }
  }

```

### <a name="LOW-3"></a>[LOW-3] `StakedUSDe._beforeTokenTransfer` can be frontran by a service.
- Scenario:
   - whenever a `BLACKLIST_MANAGER_ROLE` makes a tx, it can be searched via mempool and someone with approval of the (to be blacklisted)'s address will transfer the funds to the delegate already set.
   - Low likelyhood, should be soled when tx'ns of blacklist manager are sent to flashbots builder.
- Severity : low
- Likelihood : Med