
| |Issue|Instances|
|-|:-|:-:|
| [LOW-1](#LOW-1) | ownership transfer  in constructor is not implemented on Ownable2Step way| 1 |
| [LOW-2](#LOW-2) | transaction fails if a variant of `token` & `to`| 1 |

### <a name="LOW-1"></a>[LOW-1] `USDe::constructor` ownership transfer for first time in constructor is not implemented on Ownable2Step method

## Impact
- Initial ownership transfer should be done in Ownable2Step methof, but directly transferred in [constructor](https://github.com/code-423n4/2023-10-ethena/blob/d361ced0f57005a1b80c7d01673e17582847aaed/protocols/USDe/contracts/USDe.sol#L20)
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