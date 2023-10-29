## [NC‑1] Unnecessary operation in `StakedUSDe.sol` which consumes Gas

Function `transferInRewards` in StakedUSDe.sol has an unnecessary operation which always results in adding zero.

The problem occurs in this line:

```solidity
uint256 newVestingAmount = amount + getUnvestedAmount();
```

As the function first checks `if (getUnvestedAmount() > 0) revert StillVesting();`, the value of `getUnvestedAmount()` will always be zero at the next line. 

Code reference:
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L89-L99

Mitigation steps:
Simply delete this part `+ getUnvestedAmount()` at line 91 in the code of `StakedUSDe.sol`

