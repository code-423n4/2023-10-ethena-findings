# Unnecessary computation

## Lines of code
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L91 

## Description
There is no need to add `getUnvestedAmount()` to the amount, because if it is greater than 0, the function reverts in line 90.

## Recommendation
Remove the addition.
