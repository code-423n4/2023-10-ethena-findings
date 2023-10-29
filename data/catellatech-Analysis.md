<div align="center">
  <h1> Ethena Labs: Smart Contracts Analysis</h1>
</div>

## Index
- [Index](#index)
- [1. Ethena Labs: A Comprehensive Overview](#1-ethena-labs-a-comprehensive-overview)
  - [Security measures](#security-measures)
- [2. Main Contracts and Their Analysis Through Diagrams](#2-main-contracts-and-their-analysis-through-diagrams)
  - [Ethena uses three smart contracts:](#ethena-uses-three-smart-contracts)
    - [USDe.sol:](#usdesol)
    - [EthenaMinting.sol:](#ethenamintingsol)
    - [StakedUSDeV2.sol:](#stakedusdev2sol)
  - [Privileged Roles:](#privileged-roles)
    - [MINTER\_ROLE:](#minter_role)
    - [REDEEMER\_ROLE:](#redeemer_role)
    - [GATEKEEPER\_ROLE:](#gatekeeper_role)
    - [DEFAULT\_ADMIN\_ROLE:](#default_admin_role)
- [3. Approach taken in evaluating the codebase](#3-approach-taken-in-evaluating-the-codebase)
- [4. Ethena Labs: Architecture](#4-ethena-labs-architecture)
- [5. Systemic and Centralization Risks](#5-systemic-and-centralization-risks)
- [6. New insights and learning from this audit](#6-new-insights-and-learning-from-this-audit)
- [7. Security Approach of the Project](#7-security-approach-of-the-project)
- [8. Test analysis](#8-test-analysis)
- [9. Conclusion](#9-conclusion)

## 1. Ethena Labs: A Comprehensive Overview
**Ethena Labs** aims to create a permissionless stablecoin called **USDe**, which offers yield to users in the DeFi ecosystem. Unlike **USDC**, where Circle captures the yield, **USDe** holders can stake their **USDe** to receive **stUSDe**, which increases in value as the protocol earns yield, similar to **rETH** relative to **ETH**.

**USDe** is minted through an **RFQ process**, where users agree to a price and sign an **EIP712** signature. **stETH** is traded for newly minted **USDe** at the specified rate, and a short **ETH** perps position is created to maintain delta neutrality.

Yield generated from the long **stETH** and short **ETH** perps positions is sent to an insurance fund and staking contract.

### Security measures 
**Security measures are** in place to prevent abuse, such as limiting minting and redeeming to **100k USDe** per block, gatekeeper roles, and external organizations potentially having gatekeeper roles.

**Ethena's goal** extends beyond the technical aspects; it strives to provide users with a high degree of trust and security in the DeFi ecosystem. This involves strict control over asset freezing and repossessing, and maintaining a robust, transparent, and trustworthy operational framework.

![EthenaOverview](https://github.com/catellaTech/ethena/blob/main/EthenaOverview.drawio.png?raw=true)

## 2. Main Contracts and Their Analysis Through Diagrams
### Ethena uses three smart contracts:
#### USDe.sol: 
  - The stablecoin contract with a minter address for minting USDe.
    ![EthenaUSDe](https://github.com/catellaTech/ethena/blob/main/EthenaUSDe.drawio.png?raw=true)

#### EthenaMinting.sol: 
  - Handles the minting and redemption of USDe, primarily used by Ethena. External users provide EIP712 signatures to perform these actions.
    ![EthenaEthenaMinting](https://github.com/catellaTech/ethena/blob/main/EthenaEthenaMinting.drawio.png?raw=true)

#### StakedUSDeV2.sol: 
  - Allows USDe holders to stake and earn yield, with rewards vesting linearly over 8 hours and a cooldown period for unstaking.
    ![EthenaStakedUSDeV2](https://github.com/catellaTech/ethena/blob/main/EthenaStakedUSDeV2.drawio.png?raw=true)

### Privileged Roles:
The **EthenaMinting contract** defines some **privilege** **roles** that control who can perform specific actions within the contract:

#### MINTER_ROLE: 
This role allows the creation of **USDe** from **assets**. Holders of this role can generate new **USDe**.
  ```Solidity
  /// @notice role enabling to invoke mint
  bytes32 private constant MINTER_ROLE = keccak256("MINTER_ROLE");
  ```
#### REDEEMER_ROLE: 
Permits the **redemption** of **USDe** for **assets**. Holders of this role can **redeem USDe** for **assets**.
  ```Solidity
  /// @notice role enabling to invoke redeem
  bytes32 private constant REDEEMER_ROLE = keccak256("REDEEMER_ROLE");
  ```
 #### GATEKEEPER_ROLE: 
 This role has the power to **enable** or **disable** the ability to **mint** and **redeem** **USDe**, as well as remove minters and redeemers in emergency situations.
  ```Solidity
  /// @notice role enabling to disable mint and redeem and remove minters and redeemers in an emergency
  bytes32 private constant GATEKEEPER_ROLE = keccak256("GATEKEEPER_ROLE");
  ```
#### DEFAULT_ADMIN_ROLE: 
Allows for **administrative actions**, such as managing supported assets and custody addresses, setting minting and redeeming limits, and other general contract configurations.
  ```Solidity
    _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
  ```
> IMPORTATN ‼️‼️: The ownership of smart contracts is held in a Gnosis Safe multisig with cold wallet keys, requiring 7/10 confirmations for transactions. Additionally, there are restrictions on staking for specific addresses and the ability to freeze and repossess funds for legal and security reasons.

Given the level of control that these `roles` possess within the system, users must place full trust in the fact that the entities overseeing these `roles` will always act correctly and in the best interest of the `system` and its `users`.

## 3. Approach taken in evaluating the codebase
- High-level overview : I analyzed the overall codebase in one iteration to get a high-level understanding of the code structure and functionality.

- Documentation review : I studied the documentation to understand the purpose of each contract, its functionality, and how it is connected with other contracts.

- Literature review : I read old audits and known findings, as well as the bot races findings.

- Testing setup : I set up my testing environment and ran the tests to ensure that all tests passed. Used yarn and hardhat to test this protocol or foundry as well.

- Detailed analysis : I started with the detailed analysis of the code base, line by line. I took the necessary notes to ask some questions to the sponsors.

## 4. Ethena Labs: Architecture
The Ethena protocol comprises three main components: USDe.sol, EthenaMinting.sol, and StakedUSDeV2.sol. These components work together to facilitate the functioning of the protocol:

In general, the architecture focuses on providing a **stable token**, enabling secure minting and redemption, and offering user participation and yield opportunities. Security is a central consideration, with roles and controls thoughtfully designed to mitigate risks.

> See the following diagram to see how the users interact with the protocol

![ethenaArquitecture](https://github.com/catellaTech/ethena/blob/main/img.png?raw=true)

## 5. Systemic and Centralization Risks
- **Ethena Labs** could face some risks of **centralization** and **systemic** issues within its architecture. 

  **Centralized Control Risk**: Ethena Labs rely on a number of central roles and authorities, such as the **USDe minter**, **EthenaMinting admin**, **GATEKEEPER_ROLE**, among others. This could lead to centralized control in the hands of a few entities or individuals, which goes against the decentralized spirit of blockchain technology.

  **Censorship Risk**: The ability of **GATEKEEPER_ROLE** to disable the minting and redeeming function of **USDe** and remove roles from authorized accounts could lead to censorship issues. If these capabilities are misused or employed maliciously, users may face difficulties in accessing their assets.

  **Security Risk**: Since **Ethena Labs** uses a **role-based permission structure**, any security breach that allows malicious actors to gain access to important roles like **MINTER_ROLE** or **REDEEMER_ROLE** could have severe consequences for the platform and its users.

  **Systemic Risk**: **Ethena Labs** has a business model where it benefits from differences in asset yields. Any systemic events in the underlying markets could significantly affect the stability and performance of **USDe and stUSDe**.

Overall, **Ethena Labs** should be aware of these risks and take appropriate measures to address them and protect the interests of its users. Security, decentralization, and transparency are key considerations in the DeFi ecosystem.

## 6. New insights and learning from this audit
Our new insights from this **Ethena Labs** audit have been exponential. We have learned about the introduction of its stablecoin, **USDe**, and **unique yield** opportunities for users within its ecosystem. Key takeaways from their approach include a strong emphasis on security through **role-based controls** and risk management mechanisms. They utilize a network of **smart contracts** for **USDe minting** and **staking**, highlighting the importance of a **well-structured DeFi architecture**. User engagement is a central focus, allowing users to actively participate and earn yields. Additionally, **Ethena Labs** addresses regulatory compliance with roles tailored to specific jurisdictions. Their model underscores the complexity and challenges inherent in DeFi platforms, emphasizing the need for effective implementation and adaptability in the ever-evolving DeFi landscape.

## 7. Security Approach of the Project
What the project can add in the understanding of Security;
- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. 

- Pause Mechanism This is a chaotic situation, which can be thought of as a choice between decentralization and security.

- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ; This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. https://immunefi.com/

## 8. Test analysis
The audit scope of the contracts to be reviewed is 70%, with the aim of reaching 100% to increase the safety. 

## 9. Conclusion 
**Ethena Labs project** exhibits an interesting and well-developed architecture we believe the team has done a good job regarding the code, but the identified risks need to be addressed, and measures should be implemented to protect the protocol from potential malicious use cases. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.

### Time spent:
15 hours