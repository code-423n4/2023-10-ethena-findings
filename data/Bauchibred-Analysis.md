# Analysis
## Table of Contents

- [Approach](#approach)
- [Architecture Feedback](#architecture-feedback)
- [Centralization Risks](#centralization-risks)
- [Systemic Risks](#systemic-risks)
- [Testability](#Testability)
- [Other Recommendations](#other-recommendations)
- [Security Researcher Logistics](#security-researcher-logistics)
- [Conclusion](#conclusion)

## Approach

The audit began with a comprehensive review of the provided documentation, not limiting to just the files in scope. Following this, insights were extracted from previous audit reports over several hours. A detailed, line-by-line security assessment of the source lines of code (sLOC) ensued. The final stage involved engaging with the development team on Discord to comprehend unique implementations, discuss potential vulnerabilities, and validate critical observations.

## Architecture Feedback

- The protocol's efficiency is closely linked to its hedging strategy.
- In volatile market conditions, the protocol may fail to liquidate funds from centralized exchanges, potentially hindering users from withdrawing their full balance.
- For user redemptions, it's imperative that the protocol admin moves adequate funds from custody to the "Ethena Minting" contract.
- Concerningly, there's no on-chain guarantee for the redeemability of "USDe". The team possesses the discretion to withhold funds.
- Notable roles and their privileges:
  - **EthenaMinting & DEFAULT_ADMIN_ROLE**: 
    - Control over `maxMintPerBlock` and `maxRedeemPerBlock`.
    - Authorization to manage addresses across roles (excluding the owner).
  - **Owner**:
    - Has the power to set the `USDe` token address.
    - Can modify supported assets.
  - **MINTER_ROLE**:
    - Can generate stablecoins.
    - Holds the power to transfer any asset.
  - **REDEEMER_ROLE**:
    - Authorized to exchange stablecoins for assets.
  - **GATEKEEPER_ROLE**:
    - Ability to halt both minting and redeeming operations.
    - Can rescind the `MINTER_ROLE`.
  - For `StakedUSDeV2.sol`:
    - `Owner/DEFAULT_ADMIN_ROLE` can administer roles.
    - Can reallocate `stUSDe` from specified wallets.
    - `REWARDER_ROLE` can vest `USDe` tokens via `transferInRewards()`.

- Points of Vulnerability:
  - The lack of on-chain order pricing might allow `Ethena Labs` to mint `USDe` without the requisite underlying tokens.
  - `USDe`'s underlying assets can be extracted indiscriminately.
  - `stUSDe` tokens are susceptible to unilateral confiscation by the staking contract's administrative role.
  - Profits for `stUSDe` holders necessitate manual deposits.

- Mitigation Strategies:
  - Prioritize transparency and traceability.
  - Offer thorough user guides.
  - Design comprehensive dashboards displaying asset values and their centralized allocations.
  - Clearly define role boundaries.
  - Formulate an approved whitelist for asset custodianship.
  - Emphasize: The protocol's complexity reveals inherent centralization risks.

## Centralization Risks

- Both `maxMintPerBlock` and `_maxRedeemPerBlock` are supposed to have defined values. However, they aren't verified on-chain, allowing an admin to arbitrarily set values.
- An admin could exploit the system by assigning users the `FULL_RESTRICTED_STAKER_ROLE` and redistributing their tokens.

## Systemic Risks

- The `GATEKEEPER_ROLE` concept poses challenges. Although designed as a safeguard, an errant address could cause irreversible damage. Instead of merely revoking privileges post-factum, it's wiser to assign roles proactively based on need and then withdraw them post-usage.
- Documents and code reveal that the `EthenaMinting` contract doesn't hold underlying LST assets, which reside in custodial wallets. This structure, while possibly efficient, exposes potential liquidity challenges for `USDe` redemptions.
- The quote: "We expect external organizations to only invoke the gatekeeper functions during price discrepancies on-chain..." suggests reactive measures. A preventive approach, assigning `GATEKEEPER` roles only when necessary and revoking them afterward, would be more judicious.
- The `SOFT_RESTRICTED_STAKER_ROLE` entails partial restrictions. Addresses with this role can't deposit but can perform other functions. Such an address might transfer their `USDe` tokens elsewhere and then deposit.

## Testability

The testing coverage is admirable, with both line and statement coverage meeting standards. Nevertheless, a coverage exceeding 90% is always preferable for enhanced security.

## Other Recommendations

### **Utilize More Auditing Tools**

While the [Solidity Visual Developer](https://marketplace.visualstudio.com/items?itemName=tintinweb.solidity-visual-auditor) tool is already in use, incorporating additional plugins, especially for Visual Studio Code, could be advantageous.

### **Boost Testability**

Although the current coverage is satisfactory, striving for a 90% coverage would significantly enhance protocol reliability.

### **Optimize Event Monitoring**

The current setup could benefit from a more strategic implementation of event tracking to ensure maximum utility.

 ## Security Researcher Logistics

My attempt on reviewing the in-scope codebase spanned 12 hours distributed over 3 days:

- 0.5 hours dedicated to writing this analysis.
- 2 hours exploring the previous audit reports
- 1.5 hour were allocated for discussions with sponsors on the private discord group regarding potential vulnerabilities.
- The remaining time on finding issues and writing the report for each of them on the spot, later on _editing a few with more knowledge gained on the protocol as a whole, or downgrading them to QA reports_

## Conclusion

The codebase was a very great learning experience, though it was a pretty hard nut to crack, being that it's like an update contest and most _easy to catch_ bugs have already been spotted and mitigated from the previous contest.




### Time spent:
11 hours