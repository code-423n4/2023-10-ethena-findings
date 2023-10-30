In transferInRewards() function of StakedUSDe.sol, below code will waste gas fee unnecessarily.
```solidity
 uint256 newVestingAmount = amount + getUnvestedAmount();
```
(https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L91)
an upper line,
```solidity
 if (getUnvestedAmount() > 0) revert StillVesting(); 
```
is checking if the unvestedAmount is zero, so `newVestingAmount` should always be same with `amount`.
We can't see an another inherited function of getVestedAmount() in StakedUSDeV2.sol.

I think we can fix above line as
```solidity
 uint256 newVestingAmount = amount;
```