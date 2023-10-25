### [QA-01] Incorrect NatSpec for `addToBlacklist` and `removeFromBlacklist` functions in `StakedUSDe` contract

Relevant links:
a. https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L102
b. https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L116

#### Summary

The NatSpec of both functions state the accounts with the `DEFAULT_ADMIN_ROLE` and `BLACKLIST_MANAGER_ROLE` roles can blacklist and un-blacklist accounts, but actually only the account with the latter role can perform the task.

#### Recommendation:
Either update the functions to give direct access to the account with the `DEFAULT_ADMIN_ROLE` or update the functions NatSpec to clarify who has access to the functions.