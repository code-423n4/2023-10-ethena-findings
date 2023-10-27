# [LO-01] Missing-zero-check
...
## Proof of Concept: Detect missing zero address validation.

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDeSilo.sol#L19
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDe.sol#L23-L26

## Recommendation: Check that the address is not zero
...

# [LO-02] before-token-transfer
... 
## Proof of Concept: Follow OZ documentation using their contracts

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L245-L252

## Recommendation: Make sure that beforeTokenTransfer function is used in the correct way.
...

# [Others]
...
## Proof of Concept: `solc` frequently releases new compiler versions. Using an old version prevents access to new Solidity security checks. We also recommend avoiding complex `pragma` statement.

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDe.sol#L2
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/interfaces/ISingleAdminAccessControl.sol#L2
https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/interfaces/IUSDeDefinitions.sol#L2

## Recommendation: Deploy with any of the following Solidity versions: - 0.8.18 The recommendations take into account: - Risks related to recent releases - Risks of complex code generation changes - Risks of new language features - Risks of known bugs Use a simple pragma version that allows any of these versions. Consider using the latest version of Solidity for testing.
...
###
...
## Proof of Concept: Solidity defines a [naming convention](https://solidity.readthedocs.io/en/v0.4.25/style-guide.html#naming-conventions) that should be followed. #### Rule exceptions - Allow constant variable name/symbol/decimals to be lowercase (`ERC20`). - Allow `_` at the beginning of the `mixed_case` match for private variables and unused parameters

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/USDeSilo.sol#L15

## Recommendation: Follow the Solidity [naming convention](https://solidity.readthedocs.io/en/v0.4.25/style-guide.html#naming-conventions).
...
###
...
## Proof of Concept: Values should be assigned to variables

https://github.com/code-423n4/2023-10-ethena/blob/ee67d9b542642c9757a6b826c82d0cae60256509/contracts/StakedUSDe.sol#L184-L186
            
## Recommendation: Assign values to variables
...

