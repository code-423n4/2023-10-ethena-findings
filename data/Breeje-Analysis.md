## Summary

| Count | Topic | 
|:--:|:-------|
| 1 | Introduction |  
| 2 | Audit Approach |  
| 3 | Architecture Recommendation |  
| 4 | Centralization Risk |   
| 5 | Systemic risks |
| 6 | Time Spent |

## Introduction

This technical analysis report dig into the underlying smart contracts that constitute the stablecoin and it's staking contracts of Ethena. This report highlights insights and different Risks that User and Protocol must be aware of.

## Audit Approach

1. **Initial Scope and Documentation Review**: Thoroughly examine the Contest Readme File and project documentation to understand the protocol's objectives and functionalities.
2. **High-level Architecture Understanding**: Perform an initial architecture review of the codebase by skimming through all files without going into function details.
3. **Test Environment Setup and Analysis**: Set up a test environment and execute all tests. Additionally, use Static Analyzer tools like Slither to identify potential vulnerabilities.
4. **Comprehensive Code Review**: Conduct a line-by-line code review focusing on understanding code functionalities. 
    * **Understand Codebase Functionalities**: Begin by comprehending the functionalities of the codebase to gain a clear understanding of its operations.
    * **Analyze Value Transfer Functions**: Adopt an attacker's mindset while inspecting value transfer functions, aiming to identify potential vulnerabilities that could be exploited.
    * **Identify Access Control Issues**: Thoroughly examine the codebase for any access control problems that might allow unauthorized users to execute critical functions.
    * **Detect Reentrancy Vulnerabilities**: Look for potential reentrancy attacks where malicious contracts could exploit reentrant calls to manipulate the contract's state and behavior.
    * **Evaluate Function Execution Order**: Scrutinize the sequence of function executions to ensure the protocol's logic cannot be disrupted by changing the order of calls.
    * **Assess State Variable Handling**: Identify state variables and assess the possibility of breaking assumptions to manipulate them into exploitable states, leading to unintended exploitation of the protocol's functionality.

5. **Report Writing**: Write Report by compiling all the insights I gained throughout the line by line code review.
    

## Architecture Recommendation

`Handling of stETH can be improved`: As `stETH` is a rebasing token, it can lead to couple of issues in the `EthenaMinting` contract like:

1. `Slippage Due to Rebase Events`: Slippage in value because of rebasing when transaction is pending.
2. `Underlying Value Mismatch`: Value received by custodian addresses will always be less than intended which can break the logics used further.

I have describe about these 2 issues in detail in my QA Report.

The Architecture change I would recommend here is to use `transferShares` instead of `transferfrom` when `asset` used is `stETH`.

Reason: `stETH` balances changes upon transfers, mints/burns, and rebases, while shares balances can only change upon transfers and mints/burns. This is why it is recommended even by Lido in their [docs](https://docs.lido.fi/guides/lido-tokens-integration-guide/#steth-internals-share-mechanics) here that Use `transferShares` instead of `transferfrom` to avoid handling issues which can arise because of rebasing mechanism.

## Centralization Risk

* `EthenaMinting`: While minting `USDe`, User's `stETH` are transferred to `custodianAddress`. User needs to trust Ethena that They will be allowed to `redeem` their tokens for a fair value by giving back `USDe`. If they don't trust Ethena, they should avoid minting `USDe` as this is a potential Rug pull attack vector.

* `StakedUSDe`: Users needs to be aware of:
    * `Blacklisting`: Any user can be added to a blacklist by the Blacklist Manager, which may restrict their ability to transfer funds.
    * `redistributeLockedAmount function`: Once Blacklisted, `ADMIN` holds the power to burn User's `stUSDe` and mint it to any other address.

## Systemic risks

* The way funds are managed from `custodianAddress` will determine whether user will be able to redeem the assets at a fair valuation.

* While `stETH` is not Fees on transfer token, but transferring it through `transferFrom` will still lead to loss of few wei because the way it works is:

  1. User A transfers 1 `stETH` to User B.
  2. Under the hood, `stETH` balance gets converted to shares, integer division happens and rounding down applies using formula: `shares[account] = balanceOf(account) * totalShares / totalPooledEther`.
  3. The corresponding amount of shares gets transferred from User A to User B.
  4. Shares balance gets converted to stETH balance for User B.
  5. In many cases, the actually transferred amount is 1-2 wei less than expected.

## Time Spent

| Total Number of Hours | 18 |
|:--:|:--:|

### Time spent:
18 hours