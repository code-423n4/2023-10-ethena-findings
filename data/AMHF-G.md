Duplicate Role Checks: In both the _deposit and _withdraw functions, the contract checks if the caller and receiver have specific roles. Consider consolidating these checks into a single function or modifier to reduce gas costs and enhance code readability.

[_deposit](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L210)
[_withdraw](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L232)