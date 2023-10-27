# [N-01] Hardcoded value for `StakedUSDe::decimals `

## Description

As it can be seen in the code, the `StakedUSDe` contract has a dynamic `asset` property, that is passed into it as a constructor parameter. However, it also has `decimals` property, which is hardcoded to the value of 18. This means that if we were to deploy this contract with an ERC20 token for `asset` which has less or more than 18 decimals, the contract won't function properly. In order to mitigate this issue, the `decimals` property should either be derived from the `asset` contract, or passed in as a constructor parameter.

## Lines of code

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L185