### Advanced Analysis Report for [Ethena-Labs](https://github.com/code-423n4/2023-10-ethena)
#### Overview
- This analysis focuses on the three main contracts: [EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol), [StakedUSDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol), and [StakedUSDeV2.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol). These contracts are part of a larger ecosystem that includes [USDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol), [USDeSilo.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol), and [SingleAdminAccessControl.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol).

#### Understanding the Ecosystem:
- [EthenaMinting](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol): Responsible for minting and redeeming [USDe](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol) tokens. Implements roles and permissions.
- [StakedUSDe](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol): Allows staking of [USDe](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol) tokens. Implements blacklisting and reward distribution.
- [StakedUSDeV2](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol): Extends [StakedUSDe](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol) with cooldown functionality.

#### Codebase Quality Analysis:

**1. Structures and Libraries**
- **Mapping Structures**: The contracts use mappings for state variables. While gas-efficient, they lack the ability to iterate, making certain types of logic hard to implement.
- **SafeMath Library**: Given that Solidity 0.8.x and above have built-in overflow and underflow checks, the explicit use of SafeMath can be omitted to reduce gas costs.

**2. Modifiers and Access Control**
- **Role-Based Access Control**: The contracts use OpenZeppelin's AccessControl, which is robust but could be replaced with a more gas-efficient, in-house solution.
- **Custom Modifiers**: The names `ensureCooldownOff` and `ensureCooldownOn` could be more descriptive. Consider renaming to `requireCooldownInactive` and `requireCooldownActive`.

**3. State Variables**
- **Immutable vs Mutable**: Mutable state variables like `cooldownDuration` in [StakedUSDeV2.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol) should be better encapsulated to prevent unauthorized changes.
- **Visibility**: The choice of visibility for state variables like `cooldowns` in [StakedUSDeV2.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol) should be justified in inline comments.

**4. Events and Logging**
- **Event Emitting**: Events are emitted for significant state changes, but more could be added for better off-chain tracking.
- **Logging**: No explicit logging mechanisms are in place, which could be useful for debugging and monitoring.

**5. Function Complexity**
- **Single Responsibility**: Functions like `mint()` and `redeem()` in [EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol) perform multiple tasks, making them more complex and harder to audit.
- **Gas Costs**: Functions like `transferInRewards()` in [StakedUSDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol) could be optimized for gas by reducing state changes.

**6. Upgradability**
- **Proxy Patterns**: No upgradability patterns like proxy contracts are implemented, which could be a limitation for future upgrades.
- **Immutable Code**: Given the absence of upgradability, the code is essentially immutable once deployed, making thorough auditing crucial.

**7. Error Handling**
- **Revert Statements**: The contracts use `revert()` statements with error messages, which is a good practice for debugging.

#### Architecture Recommendations:
- Implement a DAO governance mechanism for critical parameters and role assignments.
- Consider using Chainlink oracles for any off-chain data requirements.
- Implement circuit breakers for emergency stops.
- Use a rate limiter to prevent abuse of minting and redeeming functions.

#### Centralization Risks:
- **Possible Risk**: Single admin control over key contract functionalities.
- **Potential Impact**: Admin could maliciously alter contract behavior.
- **Mitigation Strategy**: Implement multi-signature requirements or DAO-based governance.

#### Mechanism Review:
- The minting and redeeming mechanisms are well-implemented but could benefit from additional checks and balances.
- The staking mechanism in [StakedUSDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol) and [StakedUSDeV2.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol) is robust but should include more events for state changes.

#### Systemic Risks:
- Centralized control could lead to malicious activities.

#### Areas of Concern:
- Lack of upgradability could be a limitation for future improvements.
- The complexity of some functions could make them more error-prone.

#### Codebase Analysis:
## EthenaMinting Contract

### Functions and Risks

#### `mint()`
- **Specific Risk**: The function uses `nonReentrant` but still performs multiple state changes after external calls.
- **Recommendation**: Move all state changes above external calls to adhere to the Checks-Effects-Interactions pattern.

#### `redeem()`
- **Specific Risk**: The function uses a nonce and expiry for orders but doesn't account for a user sending two identical orders with different signatures.
- **Recommendation**: Include the signature in the mapping to ensure uniqueness.

#### `setMaxMintPerBlock()` and `setMaxRedeemPerBlock()`
- **Specific Risk**: These functions can be called by the admin role, potentially disrupting the contract's economics.
- **Recommendation**: Implement a timelock for sensitive admin actions.

---

## StakedUSDe Contract

### Functions and Risks

#### `transferInRewards()`
- **Specific Risk**: The function allows for the transfer of rewards but does not validate the reward amount against some maximum.
- **Recommendation**: Implement a maximum reward amount check.

#### `addToBlacklist()` and `removeFromBlacklist()`
- **Specific Risk**: The function allows blacklisting addresses, which can be abused.
- **Recommendation**: Implement a community governance mechanism for these sensitive actions.

#### `rescueTokens()`
- **Specific Risk**: This function allows the admin to withdraw any ERC20 tokens from the contract.
- **Recommendation**: Restrict this function to only allow the withdrawal of non-core tokens.

---

## StakedUSDeV2 Contract

### Functions and Risks

#### `withdraw()` and `redeem()`
- **Specific Risk**: These functions are only callable when cooldown is off, but there's no mechanism to notify or auto-toggle the cooldown.
- **Recommendation**: Implement an auto-toggle for cooldown based on certain conditions.

#### `unstake()`
- **Specific Risk**: The function allows unstaking only when the cooldown ends but doesn't account for slashing conditions.
- **Recommendation**: Implement a slashing mechanism for malicious actors.

#### `cooldownAssets()` and `cooldownShares()`
- **Specific Risk**: These functions allow for assets and shares to be put on cooldown but do not limit the number of times this can be done.
- **Recommendation**: Implement a rate limiter or maximum cooldown periods per user.

#### Recommendations:
- **Error Handling**: Implement more granular error messages for better debugging.
- **Testing**: Conduct extensive unit and integration tests, including edge cases for all functions.
- Use [Tenderly](https://dashboard.tenderly.co/) and [Defender](defender.openzeppelin.com) for continued monitoring to prevent un-foreseen risks or actions. 
- Implement a time-locked admin control for sensitive operations.
- Conduct a formal verification of the contracts.
- Implement automated testing for all contract functions.

#### Contract Details:
- Function interaction graphs I made for each contract for better visualization of function interactions:

- Link to [Graph](https://svgshare.com/i/z4w.svg) for [USDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol).

- Link to [Graph](https://svgshare.com/i/z5U.svg) for [EthenaMinting.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol).

- Link to [Graph](https://svgshare.com/i/z6A.svg) for [StakedUSDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol).

- Link to [Graph](https://svgshare.com/i/z6p.svg) for [StakedUSDeV2.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol).

- Link to [Graph](https://svgshare.com/i/z6J.svg) for [USDeSilo.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol).

- Link to [Graph](https://svgshare.com/i/z7X.svg) for [SingleAdminAccessControl.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol).

#### Conclusion:
- [Ethena-Labs](https://github.com/code-423n4/2023-10-ethena) has a well-structured and robust set of contracts, but there are areas for improvement, particularly in terms of decentralization and upgradability.

### Time spent:
15 hours