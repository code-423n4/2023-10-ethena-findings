# Analysis - Ethena Labs Contest

![Ethena Labs](https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2FLGjXgSSa9Nn.0&w=256&q=75)

## Description overview of The Ethena Labs Contest

Ethena is a synthetic dollar protocol built on Ethereum, providing a censorship-resistant and scalable crypto-native solution for money. Its stablecoin, USDe, is fully backed and composed throughout DeFi, ensuring delta-hedging and mint/redeem mechanisms for peg stability. Ethena introduces the 'Internet Bond,' combining yield from staked Ethereum and funding from perpetual and futures markets, serving as a dollar-denominated savings instrument. The goal is to offer a permissionless stablecoin, USDe, and incentivize users by allowing them to stake USDe for stUSDe, which increases in value as the protocol earns yield, similar to rETH and ETH relationship.

**The key contracts of Ethena Labs protocol for this Audit are**:

Auditing the key contracts of the Ethena Labs Protocol is essential to ensure the security of the protocol. Focusing on them first will provide a solid foundation for understanding the protocol's operation and how it manages user assets and stability. Here's why it's important to audit these specific contracts:

- **EthenaMinting.sol**: This contract have a central role in the minting and redemption processes, and it handles funds. The MINTER and REDEEMER roles are crucial for the security of the system. If an attacker compromises the MINTER role, they could potentially mint a large amount of USDe. Similarly, if REDEEMER is compromised, it could lead to a significant loss of collateral. Therefore, auditing this contract first is a priority to ensure the security of minting and redemption processes.

- **USDe.sol**: This is a stablecoin contract. It is essential to ensure that this contract correctly interacts with EthenaMinting.sol, specifically with regards to minting. Any issues in this contract could have a significant impact on your entire ecosystem. Ensure that the ownership mechanisms are secure and that the minter address is appropriately managed to prevent unauthorized minting.

- **StakedUSDeV2.sol**: This contract allows users to stake USDe and earn yield. While it's not directly involved in the minting and redemption processes, it's still a part of your financial ecosystem. You should audit it to ensure that staking, unstaking, and the yield distribution functions work as intended and that the cooldown period is correctly implemented. Be mindful of the different roles (SOFT_RESTRICTED_STAKER_ROLE and FULL_RESTRICTED_STAKER_ROLE) and their permissions.

- **USDeSilo.sol**: This contract is essential for the unstaking process and the cooldown period. You need to ensure that funds are correctly held during the cooldown and released to users as intended. Any vulnerabilities in this contract could affect the ability of users to withdraw their funds.

- **SingleAdminAccessControl.sol**: This contract is used by EthenaMinting.sol, so it's important to ensure that the access control mechanisms are correctly implemented. Any issues with this contract could affect the permissions of EthenaMinting.sol.

As the audit of these contracts, I checked for potential security vulnerabilities, such as reentrancy, access control issues, and logic flaws. Additionally, thoroughly test the functions and roles defined in these contracts to make sure they behave as expected.

## System Overview

### Scope

- **USDe.sol**: This contract is a stablecoin contract called USDe. inheriting key features from OpenZeppelin contracts. The contract allows for the minting of new tokens, but only a single approved minter can perform this action. Ownership is managed through a two-step transfer mechanism. The contract initializes with the "USDe" name and symbol, along with an initial admin address. Additionally, it enforces strict control over minting and ownership changes.

- **EthenaMinting.sol**: EthenaMinting contract is designed for minting and redeeming the "USDe" stablecoin in a single, atomic, trustless transaction. It enforces roles for minting, redemption, and emergency control. Additionally, it manages supported assets and custodian addresses. The contract sets maximum limits for minting and redemption per block to ensure security. It allows for delegation of signing authority to external accounts and ensures the validity of orders and routes. Overall, this contract provides a robust and controlled mechanism for handling the "USDe" stablecoin's minting and redemption.

- **StakedUSDe.sol**: The StakedUSDe contract enables users to stake "USDe" tokens and earn protocol rewards. It features various roles for reward distribution, blacklist management, and restricting users from staking. The contract holds a vesting mechanism for rewards and enforces a minimum shares requirement. Users can deposit, withdraw, and transfer their staked assets, with specific restrictions in place. Additionally, the contract includes functions for token rescue and redistribution of locked amounts, as well as address blacklisting and un-blacklisting.

- **StakedUSDeV2.sol**: This contract allows users to stake USDe tokens and earn rewards in the form of protocol LST and perpetual yield. The rewards are allocated to stakers based on a governance-voted yield distribution algorithm. The contract implements a cooldown mechanism, where users can stake or redeem their assets and initiate a cooldown period before claiming the underlying assets. The cooldown duration can be set by the contract owner and determines the length of the cooldown period. If the cooldown duration is set to zero, the contract follows the ERC4626 standard and disables certain functions related to cooldown. The contract also includes a silo contract (USDeSilo) that handles the withdrawal and storage of assets.

- **USDeSilo.sol**: The contract used to store USDe tokens during the stake cooldown process. The contract takes two parameters in its constructor: the address of the staking vault and the address of the USDe token. The staking vault is the only entity that is allowed to interact with this contract, as indicated by the onlyStakingVault modifier. The contract provides a withdraw function that allows the staking vault to transfer USDe tokens from the silo to a specified recipient address. SafeERC20 is used to ensure secure token transfers.

- **SingleAdminAccessControl.sol**: The SingleAdminAccessControl contract establishes a single admin role and admin transfer functionality. It uses OpenZeppelin's AccessControl for role management. Users can grant and revoke roles, except for the admin role. The contract also supports role renunciation. It enforces a transition process to change the admin role.

### Privileged Roles

Some privileged roles exercise powers over the Controller contract:

- **Admin**

```solidity
  modifier notAdmin(bytes32 role) {
    if (role == DEFAULT_ADMIN_ROLE) revert InvalidAdminChange();
    _;
  }
```

- **Owner**

```solidity
  modifier notOwner(address target) {
    if (target == owner()) revert CantBlacklistOwner();
    _;
  }
```

- **Staking-Vault**

```solidity
  modifier onlyStakingVault() {
    if (msg.sender != STAKING_VAULT) revert OnlyStakingVault();
    _;
  }
```

### Trusted Roles

      USDe minter - can mint any amount of USDe tokens to any address. Expected to be the EthenaMinting contract
      USDe owner - can set token minter and transfer ownership to another address
      USDe token holder - can not just transfer tokens but burn them and sign permits for others to spend their balance
      StakedUSDe admin - can rescue tokens from the contract and also to redistribute a fully restricted staker's stUSDe balance, as well as give roles to other addresses (for example the FULL_RESTRICTED_STAKER_ROLE role)
      StakedUSDeV2 admin - has all power of "StakedUSDe admin" and can also call the setCooldownDuration method
      REWARDER_ROLE - can transfer rewards into the StakedUSDe contract that will be vested over the next 8 hours
      BLACKLIST_MANAGER_ROLE - can do/undo full or soft restriction on a holder of stUSDe
      SOFT_RESTRICTED_STAKER_ROLE - address with this role can't stake his USDe tokens or get stUSDe tokens minted to him
      FULL_RESTRICTED_STAKER_ROLE - address with this role can't burn his stUSDe tokens to unstake his USDe tokens, neither to transfer stUSDe tokens. His balance can be manipulated by the admin of StakedUSDe
      MINTER_ROLE - can actually mint USDe tokens and also transfer EthenaMinting's token or ETH balance to a custodian address
      REDEEMER_ROLE - can redeem collateral assets for burning USDe
      EthenaMinting admin - can set the maxMint/maxRedeem amounts per block and add or remove supported collateral assets and custodian addresses, grant/revoke roles
      GATEKEEPER_ROLE - can disable minting/redeeming of USDe and remove MINTER_ROLE and REDEEMER_ROLE roles from authorized accounts

## Approach Taken-in Evaluating The Ethena Labs Protocol

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contract Overview**:

    I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality.
    The main goal was to take a close look at the important contracts and how they work together in the Ethena Labs Protocol.
    I started with the contract that is at the core of the system and from which other contracts interact or depend on. In this case, I start with the following contracts, which play crucial roles in the Ethena system:

    **Main Contracts I Looked At**

                USDe.sol
                EthenaMinting.sol
                StakedUSDeV2.sol

    I started my analysis by examining the USDe.sol contract. Begin by understanding stablecoin contract, that serves as the base currency within the Ethena system. By starting with I gain a clear understanding of the currency that all other operations and contracts revolve around. It sets the foundation for the entire ecosystem.

    Then, I turned our attention to the EthenaMinting.sol, this contract is responsible for minting and redemption processes. These are critical operations in the system, and understanding how they work is essential. after understand minting and redemption, I better grasp the flow of funds and value within the system.

    Then Dive into StakedUSDeV2.sol contract, nce I've grasped the minting and redemption processes, now I can explore the staking contract, StakedUSDeV2.sol. It allows users to stake their stablecoin and earn stUSDe. While not as foundational as USDe and EthenaMinting, it adds an additional layer to the ecosystem.

2.  **Documentation Review**:

    Then went to Review [This Docs](https://ethena-labs.gitbook.io/ethena-labs/10CaMBZwnrLWSUWzLS2a/) for a more detailed and technical explanation of Ethena protocol.

3.  **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode.

    - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

    - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

## Architecture Description and Diagram

Architecture of the key contracts that are part of the Ethena Labs protocol:

This diagram illustrates the flow for the USDe token minting/redeeming of the Ethena Labs

![Ethena-Diagram-Overview](https://i.im.ge/2023/10/30/tnFBNf.ethena1.jpg)

1. **Off-Chain User Interaction**

- A user sends a signed order to Ethena.

2. **On-Chain Ethena Actions**

- **Validation of the Order**

  Ethena validates the incoming order as follows:

  1. Calculates the hash of the order using `verifyOrder()`, resulting in a hashed order.
  2. Recovers the signer's address from the order using ECDSA and `verifyOrder()`, obtaining the signer.
  3. Validates the order:

     - If the order type is not "redeem," it reverts.
     - Ensures the signer is not the benefactor or an approved entity; otherwise, it reverts.
     - Verifies that the beneficiary address is not 0; otherwise, it reverts.
     - Checks that the collateral amount is not 0; if it is, it reverts.
     - Validates that the USDe amount is not 0; otherwise, it reverts.
     - Ensures the order has not expired; if it has, it reverts.

- **Processing Valid Orders**

  If the order is valid and not a duplicate, Ethena processes the redemption:

  1. Calls `increaseRedeemedPerBlock()` to handle the following:
     - Burns USDe tokens from the user's balance using the `burnFrom` function.
     - For asset = ETH:
       - Checks if the contract's ETH balance is less than the specified amount; if so, it reverts.
       - Transfers ETH to the user.
     - For asset = token:
       - Transfers tokens to the user using a safe transfer mechanism.

- **Handling Duplicate Orders**

  If the order is a duplicate, Ethena takes the following actions:

  1. Sets the order as invalid in storage.
  2. Verifies if the nonce used in the order is 0; if so, it reverts, as the nonce must be unique.

  This process ensures the security and validity of redemption orders while handling ETH and token transfers according to the asset type specified in the order.

### Architecture Feedback

1.  Implement multisig for admin and rewarder roles rather than single addresses. This could be a 3/5 multisig controlled by DAO members.

2.  Introduce on-chain governance for parameters like vesting periods, blacklisting rules, etc. Stakers can vote on these to distribute control.

3.  Add price feeds from multiple decentralized oracles rather than fully relying on USDe stability alone. This reduces counterparty risk, Fully off-chain components like collateral custody also introduce counterparty risks. Transparency could be improved through on-chain reserves tracking.

4.  Consider alternative collateral models that don't rely on USDe, like overcollateralized staking or basket of assets.

5.  Consider migrating to EIP-2612 standard for upgrades to introduce future improvements securely.

6.  Add on-chain decentralized governance for parameters like minter approval in USDe.sol contract.

## Codebase Quality

Overall, I consider the quality of the Ethena codebase to be excellent. The code appears to be very mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Maintainability and Reliability** | The Ethena Labs Protocol contracts demonstrates good maintainability through modular structure, consistent naming, and efficient use of libraries. It also prioritizes reliability by handling errors, conducting security checks, and using "immutable" variables. However, for enhanced reliability, consider professional security audits and documentation improvements. Regular updates are crucial in the evolving space.                                                                                                                                                                                                                                                     |
| **Code Comments**                        | The Ethena Labs codebase contains comments that briefly explain the purpose of important code elements, such as data structures, constants, and function modifiers. While these comments offer some clarity, further in-code comments are needed for individual functions and complex logic to enhance code maintainability and comprehension. Although there are some comments, adding more detailed explanations for complex functions, data structures, and key decisions can be helpful to future developers and auditors. Consider following standardized commenting conventions, such as NatSpec.                                                                             |
| **Documentation**                        | The documentation of the Ethena Labs project is quite comprehensive and detailed, providing a solid overview of how Ethena Labs is structured and how its various aspects function. However, we have noticed that there is room for additional details, such as diagrams, to gain a deeper understanding of how different contracts interact and the functions they implement. With considerable enthusiasm. We are confident that these diagrams will bring significant value to the protocol as they can be seamlessly integrated into the existing documentation, enriching it and providing a more comprehensive and detailed understanding for users, developers and auditors. |
| **Testing**                              | The audit scope of the contracts to be audited is 70% and it should be aimed to be 100%.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| **Code Structure and Formatting**        | The codebase contracts are well-structured and formatted. It inherits from multiple components, adhering for-example The USDe contract follows best practices, utilizes OpenZeppelin libraries, and ensures robust access control and error handling for safe and trustworthy operations. It's designed to be a critical part of a larger system that involves USDe and custodian addresses.The contract initializes immutable variables in the constructor.                                                                                                                                                                                                                        |

## Systemic & Centralization Risks

The analysis provided highlights several significant systemic and centralization risks present in the Ethena Labs protocol. These risks encompass concentration risk in EthenaMinting, StakedUSDe, StakedUSDeV2 risk and more, third-party dependency risk, and centralization risks arising from the existence of an “owner” role in specific contracts. However, the documentation lacks clarity on whether this address represents an externally owned account (EOA) or a contract, warranting the need for clarification. Additionally, the absence of fuzzing and invariant tests could also pose risks to the protocol’s security.

Here's an analysis of potential systemic and centralization risks in the provided contracts:

### Systemic Risks:

1. **No having, fuzzing and invariant tests could open the door to future vulnerabilities**.

2. If the assets backing the stablecoin lose value significantly (e.g. due to market crash), it could cause a run on the stablecoin and collapse of its peg. This could spread risk to other DeFi protocols using USDe.

3. If the stablecoin becomes widely used in DeFi but later fails or exits, it could cause liquidity issues and losses across many applications.

4. A compromise of the admin, gatekeeper or minter roles would allow unauthorized minting/redeeming, threatening stability

5. StakedUSDe.sol contract relies on users staking USDe tokens. If a significant number of users decide to stake or unstake their tokens simultaneously, it may lead to potential issues such as front-running, network congestion, and gas price spikes and the contract depends on rewards being distributed from another contract (the rewarder). Delays or issues in rewards distribution can affect the staking and unstaking experience for users, potentially leading to dissatisfaction or withdrawal of staked assets.

6. In StakedUSDe.sol contract includes a vesting period for rewards. If the vesting period is too long or if it's not properly enforced, it can result in users feeling like their rewards are locked up for an extended period.

### Centralization Risks:

1. The contract USDe.sol allows only a single approved minter to mint new tokens. This introduces a significant centralization risk because the entire minting process depends on this single entity. If the minter's private key is compromised, it could lead to unauthorized minting of tokens, affecting the stability and trustworthiness of the system and While not visible in the provided code, if there are custodian addresses involved in managing assets, the centralization risk increases. Custodians hold significant power over the assets, and their compromise or mismanagement can impact the stability of the stablecoin.

2. Off-chain custodians hold the backing assets in EthenaMinting contract, but there is no on-chain transparency into reserves or attestations of their backing. Users must fully trust custodians.

3. In this contract EthenaMinting the ability for contracts to delegate signing to EOA addresses could introduce centralization risk, as it allows external addresses to act on behalf of the contract. Care must be taken to ensure that only trusted entities can delegate.

4. The REWARDER_ROLE can control the distribution of rewards to the contract in StakedUSDe.sol. The initial rewarder and any subsequent rewarder role holders have significant control over the rewards allocated to stakers, potentially centralizing power in the hands of a few entities and the contract can restrict transfers and staking for certain addresses based on roles. While this feature can be used for security and compliance reasons, it also centralizes the decision-making process.

**Properly managing these risks and implementing best practices in security and decentralization will contribute to the sustainability and long-term success of the Ethena Labs protocol.**

## Conclusion

In general, the Ethena Labs project exhibits an interesting and well-developed architecture we believe the team has done a good job
regarding the code, but the identified risks need to be addressed, and measures should be implemented to protect the protocol from
potential malicious use cases. Additionally, it is recommended to improve the documentation and comments in the code to enhance
understanding and collaboration among developers. It is also highly recommended that the team continues to invest in security
measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.


### Time spent:
12 hours