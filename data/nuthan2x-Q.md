## `USDe::constructor` ownership transfer for first time in constructor is not implemented on Ownable2Step method

## Impact
- Initial ownership transfer should be done in Ownable2Step methof, but directly transferred in [constructor](https://github.com/code-423n4/2023-10-ethena/blob/d361ced0f57005a1b80c7d01673e17582847aaed/protocols/USDe/contracts/USDe.sol#L20)
- First call `transferOwnership(admin)` then from admin key, call `acceptOwnership()`
- Severity : Low
- Likelihood : Medium

## Proof of Concept
NA

## Tools Used
- Manual verification

## Recommended Mitigation Steps
```diff
-  _transferOwnership(admin);
+  transferOwnership(admin);
```