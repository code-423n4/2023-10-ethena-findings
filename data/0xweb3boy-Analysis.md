# Ethena Labs - Enabling The Internet Bond
Audit contest: `Ethena Labs`

## Table of Contents

- 1. Executive Summary
- 2. Code Audit Approach
  - 2.1 Audit Documentation and Scope
  - 2.2 Code review
  - 2.3 Threat Modelling
  - 2.4 Exploitation and Proofs of Concept
  - 2.5 Report Issues
- 3. Architecture overview
  - 3.1 Overview of Open Dollar as a Lending Protocol 
  - 3.2  Key Contracts Introduced
  - 3.3 Scope of the Audit
- 4. Implementation Notes
  - 4.1 General Impressions
  - 4.2 Composition over Inheritance
  - 4.3 Comments
  - 4.4 Solidity Versions
- 5. Conclusion

## 1. Executive Summary

In focusing on the ongoing audit contest Ethena Labs, my analysis starts by delineating the code audit methodology applied to the contracts within the defined scope. Subsequently, I provide insights into the architectural aspects, offering my perspective. Finally, I offer observations pertaining to the code implementation.


I want to emphasize that unless expressly specified, any potential architectural risks or implementation concerns discussed in this document should not be construed as vulnerabilities or suggestions to modify the architecture or code based solely on this analysis. As an auditor, I recognize the necessity of a comprehensive evaluation of design choices in intricate projects, considering risks as only one component of a larger evaluative process. It's essential to acknowledge that the project team may have already evaluated these risks and established the most appropriate approach to mitigate or coexist with them.

## 2. Code Audit Approach

Time spent: 18 hours

### 2.1 Audit Documentation and Scope
Commencing the analysis, the first phase entailed a thorough review of the https://github.com/code-423n4/2023-10-ethena to fully grasp the fundamental concepts and limitations of the audit. This initial step was crucial in guiding the prioritization of my audit efforts.


### 2.2 Code review


Initiating the code review, the starting point involved gaining a comprehensive understanding of 

## EthenaMinting.sol 
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol 
## StakedUSDeV2.sol
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol
## USDe.sol
https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol

all these are a pivotal components responsible for implementing Ethena Labs. This understanding of the core pattern significantly facilitated the comprehension of the protocol contracts and their interconnections. During this phase, meticulous documentation of observations and the formulation of pertinent questions regarding potential exploits were undertaken, striking a balance between depth and breadth of analysis.

### 2.3 Threat Modelling

The initial step involved crafting precise assumptions that, if breached, could present notable security risks to the system. This approach serves to guide the identification of optimal exploitation strategies. Although not an exhaustive threat modeling exercise, it closely aligns with the essence of such an analysis.

### 2.4 Exploitation and Proofs of Concept

Progressing from this juncture, the primary methodology took the form of a cyclic process, conditionally encompassing steps 2.2, 2.3, and 2.4. This involved iterative attempts at exploitation and the subsequent creation of proofs of concept, occasionally aided by available documentation or the helpful community on Discord. The key focus during this phase was to challenge fundamental assumptions, generate novel ones in the process, and refine the approach by utilizing coded proofs of concept to hasten the development of successful exploits.

### 2.5 Report Issues

While this particular stage might initially appear straightforward, it harbors subtleties worth considering. Hastily reporting vulnerabilities and subsequently overlooking them is not a prudent course of action. The optimal approach to augment the value delivered to sponsors (and ideally, to auditors as well) entails thoroughly documenting the potential gains from exploiting each vulnerability. This comprehensive assessment aids in the determination of whether these exploits could be strategically amalgamated to generate a more substantial impact on the system's security. It's important to recognize that seemingly minor and moderate issues, when skillfully leveraged, can compound into a critical vulnerability. This assessment must be weighed against the risks that users might encounter. Within the realm of Code4rena audit contests, a heightened level of caution and an expedited reporting channel are accorded to zero-day vulnerabilities or highly sensitive bugs impacting deployed contracts.

## 3. Architecture overview


### 3.1 Overview of Ethena Labs 

Ethena aims to provide a decentralized stablecoin called USDe to the DeFi community, enabling users to earn yields by participating in the ecosystem. In contrast to USDC, where yield accrues to Circle, USDe holders can stake their tokens to receive stUSDe, which appreciates in value compared to USDe as the protocol generates yield, akin to how rETH's value increases in relation to ETH.


Ethena offers a process where users can inquire about the amount of USDe they can create using a specific amount of stETH. If a user agrees to the provided exchange rate, they validate the transaction by signing an EIP712 signature and sending it to Ethena. Following this, stETH is swapped for newly generated USDe at the rate indicated by Ethena's inquiry, all while the user's signature is in place. The stETH is then moved to custodians and managed on a perpetual exchange platform, where an equivalent USD amount of ETH perps is shorted to create a balanced financial position.


Minting USDe using stETH results in Ethena taking a corresponding short position in ETH perps on perpetual exchanges. stETH offers an annualized yield of 3-4%, while short ETH perps provide a yield of 6-8%. The combined daily yield from the long and short positions is funneled into an insurance fund and subsequently transferred to the staking contract every 8 hours.


### 3.2 Key Contracts Introduced:

#### EthenaMinting.sol

> EthenaMinting.sol is the contract with an associated address referenced by the minter variable in USDe.sol.
> Users minting USDe with stETH or redeeming collateral for USDe trigger the execution of this contract.
> The key functions used within this contract are "mint()" and "redeem()".
> All users interacting with this contract are part of Ethena's ecosystem.
> When external users want to mint or redeem, they generate an EIP712 signature based on offchain pricing provided by Ethena.
> These external users sign the order and transmit it back to Ethena's backend.
> Ethena's backend conducts various validation checks on the signed order and is responsible for placing these orders on the blockchain.
> Ethena's architecture is designed in a way that restricts the execution of "mint()", "redeem()", and other functions in this contract to be exclusively performed by Ethena.

#### StakedUSDeV2.sol
The StakedUSDeV2.sol contract allows USDe stablecoin holders to stake their tokens, receive stUSDe in return, and earn yield. This contract modifies the ERC4626 standard by vesting rewards linearly over 8 hours to prevent users from front-running yield payments. It also introduces a 14-day cooldown period for unstaking stUSDe, where the user's stUSDe is burnt immediately, and they can withdraw USDe after the cooldown period. A separate silo contract holds the funds during this period, and users can configure the cooldown duration up to 90 days.

For legal compliance, the contract has two roles: SOFT_RESTRICTED_STAKER_ROLE for users in restricted countries who can't deposit or withdraw directly but can participate in yield through open market trading, and FULL_RESTRICTED_STAKER_ROLE for more severe restrictions, allowing Ethena to freeze and repossess funds. This flexibility aims to balance security with user trust, considering the tie-in with centralized finance for yield generation. It's important to note that these restrictions only apply to the staking contract and do not affect the USDe stablecoin, which remains unrestricted, unlike USDC.


### 3.3 Scope of the Audit

| Contract |SLOC  |Purpose |
|:--|:----------------|:------|
|[USDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDe.sol) |24 | USDe token stablecoin contract that grants another address the ability to mint USDe |
|[USDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/EthenaMinting.sol) |295 | The contract where minting and redemption occurs. USDe.sol grants this contract the ability to mint USDe  |
|[StakedUSDe.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDe.sol) |150 | Extension of ERC4626. Users stake USDe to receive stUSDe which increases in value as Ethena deposits protocol yield here |
|[StakedUSDeV2.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/StakedUSDeV2.sol) |76 | Extends StakedUSDe, adds a redemption cooldown. |
|[USDeSilo.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/USDeSilo.sol) |20 |Contract to temporarily hold USDe during redemption cooldown |
|[SingleAdminAccessControl.sol](https://github.com/code-423n4/2023-10-ethena/blob/main/contracts/SingleAdminAccessControl.sol) |43 | EthenaMinting uses SingleAdminAccessControl rather than the standard AccessControl |



## 4. Implementation Notes

During the course of the audit, several noteworthy implementation details were identified, and among these, a significant subset holds potential value for the ongoing analysis.

### 4.1 General Impressions

# Overview of Ethena's Architecture

Ethena's Goals:

Ethena's objective is to provide a permissionless stablecoin called USDe to DeFi users while rewarding them for participating in the ecosystem.
Unlike USDC, where Circle captures the yield, USDe holders can stake their tokens to receive stUSDe, which appreciates in value as the protocol generates yield, similar to rETH's relationship with ETH.
Minting USDe:

Ethena facilitates the minting of USDe by offering an RFQ mechanism for users to determine how much USDe they can create using stETH or other collateral.
Users who agree to the provided price sign an EIP712 signature and submit it to Ethena. This triggers the exchange of stETH for newly minted USDe at the specified rate.
Ethena then shorts an equivalent amount of ETH perps on a perpetual exchange, creating a delta-neutral position.
Earning Yield:

Ethena generates yield by minting USDe with stETH and simultaneously opening a short ETH perps position on perps exchanges.
stETH provides an annualized yield of 3-4%, while short ETH perps yield 6-8%.
The combined yield from these positions is directed to an insurance fund and subsequently sent to the staking contract every 8 hours.
Delta Neutrality:

The long stETH and short ETH perps positions are designed to maintain a fixed value at the time of creation.
If the market experiences a significant downturn, the user can redeem USDe, and the short perps position can be closed to realize profits and obtain stETH, restoring the user's position to its original value.


### 4.2 Inheritance over Composition

Ethena Labs is using Inheritance over composition, Inheritance in software design relies on a hierarchical structure where classes or objects inherit properties and behavior from a parent class or base object. While inheritance offers simplicity and a hierarchical organization, it often lacks the flexibility and adaptability found in composition. Inheritance can lead to a rigid code structure where changes in the base class may impact all derived classes, potentially creating a tightly coupled system that is challenging to maintain and test.

Moreover, inheritance can present challenges like the diamond inheritance problem, where multiple classes inherit from a common base, leading to ambiguity and complexities in method resolution. While inheritance can promote code reuse to some extent, it tends to bind components more closely and may not allow for the dynamic assembly of functionality that composition enables.

Overall, inheritance can provide a straightforward and organized structure but may lack the versatility, scalability, and modularity offered by composition. The choice between composition and inheritance often depends on the specific requirements of a software system and the trade-offs between simplicity and adaptability.


### 4.3 Comments

#### Importance of Comments for Clarity:

Comments serve a pivotal role in enhancing the understandability of the codebase. While the code is generally clean and logically structured, judiciously placed comments can provide valuable insights into the functionalities and intentions behind the code. They contribute to a better comprehension of the code's purpose, especially for auditors and developers involved in the analysis.

#### Strategic Comment Placement:

The codebase would greatly benefit from an increased presence of comments. Strategic placement of comments within the essential contracts can significantly aid auditors in comprehending the implementation details. These comments should elucidate the logic, processes, and methodologies employed, promoting a seamless audit experience.

#### Facilitating Audits and Code Readability:

Comprehensive comments in the all of the contracts can expedite the auditing process by allowing auditors to swiftly grasp the intended functionality of the code. This, in turn, enhances the readability of the functions and methods, making it easier for auditors to identify any inconsistencies or deviations between the documented intentions and the actual code implementation.

#### Detecting Discrepancies in Code Intentions:

An important aspect of code review is discerning any mismatches between the documented intentions in the comments and the actual code implementation. Comments that accurately reflect the code's purpose are crucial for auditors, as discrepancies between the two can be indicators of potential vulnerabilities or errors. The act of aligning comments with the true code behavior is fundamental to ensuring the reliability and security of the smart contract.


### 4.4 Solidity Versions

Though there are valid arguments both in support of and against adopting the latest Solidity version, I find that this discussion bears little significance for the current state of the project. Without a doubt, choosing the most up-to-date version is a far superior decision when compared to the potential risks associated with outdated versions.


## 5. Conclusion

### Positive Audit Experience:

The process of auditing this codebase and evaluating its architectural choices has been thoroughly enjoyable and enriching. Navigating through the intricacies and nuances of the project has been enlightening, presenting an opportunity to delve into the complexities of the system.

### Strategic Simplifications for Complexity Management:

Inherent complexity is a characteristic of many systems, especially those in the realm of blockchain and smart contracts. The strategic introduction of simplifications within this project has proven to be a valuable approach. These simplifications are well-thought-out and strategically implemented, demonstrating an understanding of how to manage complexity effectively.

### Achieving a Harmonious Balance:

A notable achievement of this project is striking a harmonious balance between the imperative for simplicity and the challenge of managing inherent complexity. This equilibrium is crucial in ensuring that the codebase remains comprehensible, maintainable, and scalable, even as the system becomes more intricate.

### Importance of Methodology Overview:

The overview provided regarding the methodology employed during the audit of the contracts within the defined scope is invaluable. It offers a clear and structured insight into the analytical approach undertaken, shedding light on the depth and rigor of the evaluation process.

### Relevance for Project Team and Stakeholders:

The insights presented are not only beneficial for the project team but also extend to any party with an interest in analyzing this codebase. The detailed observations, considerations, and recommendations have the potential to guide and inform decision-making, contributing to the project's overall improvement and security.


### Time spent:
18 hours


### Time spent:
18 hours