# 🕵️‍♀️ Analysis - Ethena Labs

## Summary

| List | Head                                            | Details                                                                                |
| :--- | :---------------------------------------------- | :------------------------------------------------------------------------------------- |
| a)   | The approach I followed when reviewing the code | Stages in my code review and analysis                                                  |
| b)   | Analysis of the code base                       | What is unique? How are the existing patterns used?                                    |
| c)   | Test analysis                                   | Test scope of the project and quality of tests                                         |
| d)   | Centralization risks                            | How was the risk of centralization handled in the project, what could be alternatives? |
| e)   | Security Approach of the Project                | Audit approach of the Project                                                          |
| f)   | Other Audit Reports and Automated Findings      | What are the previous Audit reports and their analysis                                 |
|      |

## a) The approach I followed when reviewing the code

Determine the scope of code audit:
https://github.com/code-423n4/2023-10-ethena

| Number | Stage                | Details                                                                                                             | Information                                                                      |
| :----- | :------------------- | :------------------------------------------------------------------------------------------------------------------ | :------------------------------------------------------------------------------- |
| 1      | Compile and Run Test | [Installation](https://github.com/code-423n4/2023-10-ethena)                                                        | Test and installation structure is simple, cleanly designed                      |
| 2      | Architecture Review  | https://ethena-labs.gitbook.io/ethena-labs/10CaMBZwnrLWSUWzLS2a/solution-design/overview                            | Understand the functions of each module                                          |
| 3      | Graphical Analysis   | Graphical Analysis with [vscode-solidity-auditor](https://github.com/Consensys/vscode-solidity-auditor)             | Solidity language support and visual security auditor for Visual Studio Code     |
| 4      | Test Suits           | [Tests](https://github.com/code-423n4/2023-10-ethena/tree/main/test)                                                | In this section, the scope and content of the tests of the project are analyzed. |
| 5      | Code Review          | [Scope](https://github.com/code-423n4/2023-10-ethena/tree/main#scope)                                               | Top-down analysis of codes according to architectural design, IDE used: VsCode   |
| 6      | What Is Ethena?      | [Ethena](https://mirror.xyz/0xF99d0E4E3435cc9C9868D1C6274DfaB3e2721341/2gfr0qaFvZ8UxPaBvPPAZgwdcbssR_cyg5svqj1YGrY) |                                                                                  |
| 7      | Infographic          | [Excalidraw](https://excalidraw.com/)                                                                               | I made Visual drawings to understand the hard-to-understand mechanisms           |

## b) Analysis of the code base

![Ethena](https://raw.githubusercontent.com/0x7869616F/img/main/Ethena.png)

## c) Test analysis

### Fork https://github.com/code-423n4/2023-10-ethena/

### USDe.t.sol

Initialization of the contract: By setting different private keys, the initial settings of the contract are simulated, including the owner, new owner, minter, etc.

Owner Transfer: Tested the transfer of contract owners to ensure that only the current owner can initiate this operation.

Minting function: The minting function of the contract was tested and verified whether different addresses can successfully mint stablecoins (USDe).

Permission control: The permission control mechanism in the contract was tested to ensure that only specific roles (such as minters) can perform corresponding operations.

### EthenaMinting.core.t.sol

Testing of minting and redemption functions to ensure they work as expected.

Verify that a rollback is triggered when the contract receives an unsupported asset (ERC-20 token or Ethereum).

Check the validity period of the order to ensure that a rollback is triggered when the order expires.

Tests for adding and removing supported assets to ensure these operations can be performed successfully.
Verify whether an error in sending a redemption order to the mint function or a mint order to the redeem function will trigger a rollback.

Test sending ether to the contract to make sure the contract can receive ether.

### EthenaMinting.blockLimits.t.sol

#### Maximum minting limit test

`test_multiple_mints` tests multiple mint operations to ensure that the number of mints per block gradually increases but does not exceed the maximum mint limit.

`test_fuzz_maxMint_perBlock_exceeded_revert` simulates the contract triggering a rollback when the amount of mint exceeds the maximum mint limit.

`test_fuzz_mint_maxMint_perBlock_exceeded_revert` tests whether a rollback will be triggered if the maximum mint limit is exceeded during the minting operation.

`test_fuzz_nextBlock_mint_is_zero` ensures that in the next block, the maximum mint limit is reset to zero.

`test_fuzz_maxMint_perBlock_setter` tests the ability to set the maximum mint limit.

#### Maximum redemption limit test

`test_multiple_redeem` tests multiple redemption operations to ensure that the number of redemptions in each block gradually increases but does not exceed the maximum redemption limit.

`test_fuzz_maxRedeem_perBlock_exceeded_revert` simulates the contract triggering rollback when the number of redemptions exceeds the maximum redemption limit.

`test_fuzz_nextBlock_redeem_is_zero` ensures that in the next block, the maximum redemption limit is reset to zero.

`test_fuzz_maxRedeem_perBlock_setter` tests the ability to set the maximum redemption limit.

### EthenaMinting.Delegate.t.sol

`testDelegateSuccessfulMint`: This test case verifies a successful delegate mint operation. First, benefactor sets the proxy signer to trader2. Benefactor then creates an order, trader2 signs the order with its own private key, and performs the minting operation. Finally, the test checks that post-minting balances are distributed correctly.

`testDelegateFailureMint`: This test case verifies minting failure of proxy signatures. In this case, benefitfactor has no proxy signer set and trader2 attempts to perform a minting operation, but should fail. The test case checks whether the operation failed and verifies that the balance has not been changed.

`testDelegateSuccessfulRedeem`: This test case verifies a successful delegate redemption operation. It is similar to testDelegateSuccessfulMint, but performs a redemption operation.

`testDelegateFailureRedeem`: This test case verifies the redemption failure of the proxy signature, similar to testDelegateFailureMint, but performs a redemption operation.

`testCanUndelegate`: This test case verifies that proxy signing can be canceled. It first sets the proxy signer to trader2 and then deletes it. Next, trader2 attempts to perform a minting operation, but since the proxy signature has been cancelled, the operation should fail.

### StakedUSDe.t.sol

Staking Tests:

        Checking the initial stake, staking amount below the minimum, ensuring insufficient shares for withdrawal, and unstaking.

Reward Distribution:

        Tests ensuring only the rewarder can send rewards and covers the stake before and after the reward is provided.

Time-based and Vested Tests:

        Ensures the time-based vesting mechanism, rewards transfer, and vested amounts under different time scenarios.

Owner Operations:

        Validates the owner's permissions to rescue tokens, change rewarder, and ensure token conversions.

Fair Staking and Unstaking Prices:

        Tests checking the fairness in staking and unstaking operations for different users.

Fuzz Testing:

        A set of tests to simulate various scenarios, especially with significant amounts, to evaluate the staking, reward, and vesting mechanisms under varying conditions.

Miscellaneous Tests:

        Checking decimals, mint functions, transfers, and scenarios related to the donation attack.

### StakedUSDe.blacklist.t.sol

Role Management:

    Roles like 'FULL_RESTRICTED_STAKER_ROLE' and 'SOFT_RESTRICTED_STAKER_ROLE' are defined, and the contract restricts user actions based on these roles.
    There's a 'BLACKLIST_MANAGER_ROLE' that is responsible for blacklisting and unblacklisting addresses.

Address Blacklisting:

    The tests cover blacklisting, unblacklisting, and the inability to blacklist the contract owner.
    It also includes scenarios where a 'BLACKLIST_MANAGER_ROLE' can add or remove blacklisted addresses.

Access Control:

    The tests ensure that the 'DEFAULT_ADMIN_ROLE' cannot be revoked and that 'BLACKLIST_MANAGER_ROLE' actions are restricted.

Specific Scenarios:

    There are specific scenarios where actions such as burning tokens, transferring to the zero address, and other restricted actions are tested.

Role Revocation:

    The code verifies that unauthorized revocation of roles, such as by the contract owner or the account itself, should fail.

Ownership Management:

    The owner can remove the 'BLACKLIST_MANAGER_ROLE' from an address.

### StakedUSDeV2.blacklist.t.sol

User Staking and Redemption Flow Tests:

    It tests the typical staking and redemption flow for a common user.Users deposit assets, redeem them, and the balances are checked.

Soft and Full Blacklist Tests:

    It tests how the contract behaves when users are added to the soft blacklist or full blacklist.
    Deposits, withdrawals, and transfers are tested under these conditions.
    The ability of users on the blacklist to transfer assets to others is also examined.

Redistribution of Locked Amount Test:

    It tests the redistribution of locked amounts from one user to another.

Access Control Tests:

    It tests the behavior of access control features, including role granting and revoking.
    It checks that certain actions can only be performed by specific roles (e.g., BLACKLIST_MANAGER_ROLE).

Additional Tests:

    Various other scenarios and edge cases are tested, such as users attempting to revoke their own roles, trying to grant roles to others when they don't have the authority, and ensuring that the contract owner can't renounce their role.

Owner Removing Blacklist Manager:

    It tests the ability of the contract owner to remove the BLACKLIST_MANAGER_ROLE from a user.

### StakedUSDeV2.cooldownEnabled.t.sol

test_constructor:

    Checks the initialization of the StakedUSDeV2 contract to verify the owner, cooldown duration, and the silo address.

\_mintApproveDeposit:

    Tests the minting, approval, and depositing process, verifying that an account can deposit tokens into the contract.

\_redeem:

    Tests the cooldown and unstaking process, allowing users to redeem their assets after a cooldown period.

\_transferRewards:

     Checks the ability to transfer rewards to the staking contract, updating the vested amount accordingly.

\_assertVestedAmountIs:

     Verifies that the vested amount in the staking contract matches the expected value.

testStakeUnstake:

    Tests the staking and unstaking process for users, ensuring that assets are correctly transferred.

testOnlyRewarderCanReward:

    Ensures that only the designated rewarder can transfer rewards to the contract.

testStakingAndUnstakingBeforeAfterReward:

    Tests the staking and unstaking processes before and after reward transfers.

testFuzzNoJumpInVestedBalance:

    Tests the vesting process and checks for changes in vested amounts over time.

testOwnerCannotRescueUSDe:

     Checks that the owner cannot rescue USDe tokens from the staking contract.

testOwnerCanRescueUSDe:

     Verifies that the owner can rescue USDe tokens from the staking contract.

testOwnerCanChangeRewarder:

    Tests the ability of the owner to change the rewarder address.

testUSDeValuePerStUSDe:

    Checks the calculation of USDe value per StakedUSDe and ensures it works as expected.

testFairStakeAndUnstakePrices:

    Verifies the fairness of stake and unstake prices for different users.

testFuzzFairStakeAndUnstakePrices:

     Fuzz testing for fair stake and unstake prices under various conditions.

testTransferRewardsFailsInsufficientBalance:

    Ensures that transferring rewards fails when the rewarder's balance is insufficient.

testTransferRewardsFailsZeroAmount:

    Tests that transferring rewards with a zero amount fails.

testDecimalsIs18:

    Checks that the decimals setting for the contract is 18.

testMintWithSlippageCheck:

    Tests minting with slippage checks.

testMintToDiffRecipient:

    Verifies that minting to different recipients works as expected.

testFuzzCooldownAssetsUnstake:

    Tests the cooldown and unstaking of assets with varying amounts.

test_fails_v1_exit_functions_cooldownDuration_gt_0:

    Ensures that version 1 exit functions fail when the cooldown duration is greater than zero.

test_fails_v2_if_set_duration_zero:

    Verifies that certain functions in version 2 fail when the cooldown duration is set to zero.

testFuzzCooldownAssets:

    Tests the cooldown and asset unstaking process under various conditions.

testFuzzCooldownShares:

    Tests the cooldown and shares unstaking process under various conditions.

testSetCooldown_zero:

    Tests the setting of a cooldown period to zero.

testSetCooldown_error_gt_max:

    Verifies that setting a cooldown period greater than the maximum allowed duration results in an error.

testSetCooldown_fuzz:

    Tests the setting of various cooldown durations for the staking contract.

### StakedUSDeV2.cooldownDisabled.t.sol

test_cooldownShares_fails_cooldownDuration_zero:

    This test verifies that the cooldownShares function fails when the cooldown duration is set to zero in StakedUSDeV2. It's a test for the expected behavior.

test_cooldownAssets_fails_cooldownDuration_zero:

    This test ensures that the cooldownAssets function fails when the cooldown duration is set to zero in StakedUSDeV2. It's a test for the expected behavior.

## d) Centralization risks

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L162C1-L167C44

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L194-L198

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L247

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L270

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L283

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L290

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol#L290

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L23

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol#L23

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol#L26-L32

https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol#L126

## e) Security Approach of the Project

ccess Control:

Use role-based access control (RBAC) to restrict functions and access permissions to specific roles or addresses. This approach is already implemented in the project, such as the "onlyStakingVault" modifier in the USDeSilo contract.

Input Validation:

Implement input validation and sanitization to prevent invalid or malicious inputs from being processed by the smart contracts. Validate all inputs to functions, including addresses and amounts.

Use Reentrancy Guard:

Utilize the OpenZeppelin ReentrancyGuard contract, as demonstrated in the StakedUSDe contract, to prevent reentrancy attacks.

Emergency Stop Mechanism:

Consider implementing an emergency stop mechanism, which allows pausing or halting the contract's critical functions in case of unexpected vulnerabilities or attacks.

Token Transfer Safety:

Use the SafeERC20 library for token transfers to prevent common vulnerabilities such as overflows, underflows, and reentrancy attacks.

Minter Control:

Maintain strict control over the minter address to ensure that only authorized addresses can create new tokens. Be cautious about who has access to the minter role.

Upgradeability:

Consider the implications of contract upgradeability. Any changes to contract functionality or logic should be performed with caution and follow transparent upgrade mechanisms if necessary.

Security Documentation:

Develop comprehensive security documentation that includes threat models, security requirements, and responses to potential security incidents.

Continuous Monitoring:

Implement continuous monitoring and testing procedures to identify and address security issues as they emerge.

Community Involvement:

Encourage the community to participate in the security process by offering bug bounties or rewards for responsible disclosure of vulnerabilities.

## f)Other Audit Reports

**Automated Findings:**
https://github.com/code-423n4/2023-10-ethena/blob/main/bot-report.md

**Other Audit Reports:**
https://github.com/code-423n4/2023-10-ethena/tree/main/audit

## Time spent

35 hours






### Time spent:
35 hours