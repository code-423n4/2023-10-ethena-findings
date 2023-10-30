# Ethena Labs - Analysis 

|Head |Details|
|:----------------|:------|
|Description| A brief introduction to the protocol
| Approach taken in evaluating the codebase | What is unique? How are the existing patterns used? |
|Codebase quality analysis| Its structure, readability, maintainability, and adherence to best practices|
|Architecture recommendations| The design of the protocol|
|Centralization risks| power, control, or decision-making authority is concentrated in a single entity|
|Other recommendations| Recommendations for improving the quality of your codebase|
|Conclusion| Takeaway from this protocol, recommendations and final review|
|Time spent| Total time spent during auditing and reviewing the codebase |

## Description
The Ethena protocol is building USDe which will be a synthetic dollar with yield bearing properties, deployed on Ethereum. The stablecoin will be 100% collateralized with no collateral within the banking system, using as collateral USDC, stETH and other LSDs. The yield is expected to come from stETH and arbitrage. The USDe smart contract's minting and redeeming will be handled in a trusted manner by the Ethena team.

The key contracts of the protocol for this Audit are:

1. ``USDe.sol`` : This is the contract of the stablecoin. It extends ERC20Burnable, ERC20Permit and Ownable2Step from Open Zepplin. There's a single variable, the minter address that can be modified by the OWNER. Outside of Ownable2Step contract owner only has one custom function, the ability to set the minter variable to any address.

2. ``EthenaMinting.sol``: This is the contract and address that the minter variable in USDe.sol points to. When users mint USDe with stETH (or other collateral) or redeems collateral for USDe, this contract is invoked. The primary functions used in this contract is mint() and redeem().

3. ``StakedUSDeV2.sol``: This is where holders of USDe stablecoin can stake their stablecoin, get stUSDe in return and earn yield. The protocol's yield is paid out by having a REWARDER role of the staking contract send yield in USDe, increasing the stUSDe value with respect to USDe.


## Approach taken in evaluating the codebase

Steps:

- ``Using a static code analysis tool``: Static code analysis tools can scan the code for potential bugs and vulnerabilities. These tools can be used to identify a wide range of issues, including:

    - Insecure coding practices
    - Common vulnerabilities
    - Code that is not compliant with security standards

- ``Reading the documentation``: The documentation was perfectly crafted by the Ethena Labs team explaining all the necessary components and mechanisms of the code base with examples. IT
t helped to provide a detailed overview of the protocol and its code base. It also helped to to understand the purpose of the code and to identify potential areas of concern.

- ``Scoping the analysis``: Once you have a basic understanding of the protocol and its codebase, you can start to scope the analysis. This involves identifying the specific areas of code that you want to focus on. For example, you may want to focus on the code that handles user input, the code that interacts with external calls, or the code that changes sensitive state.

- ``Manually reviewing the code``: Once you have scoped the analysis, you can start to manually review the code. This involves reading the code line-by-line and looking for potential problems. Some of the things you should look for include:

   - Unvalidated user input
   - Unsafe external calls
   - Unexpected state changes
   - Lack of Access Control

- ``Marking vulnerable code parts with @audit tags``: Once you have identified any potential vulnerabilities, you should mark them with @audit tags. This will help you to identify the vulnerable code parts later on. 

- ``Digging deep into vulnerable code parts and compare with documentations``:  For each vulnerable code part, you should dig deep to understand how it works and why it is vulnerable. You should also compare the code with the documentation to see if there are any discrepancies.

- ``Performing a series of tests``: Once you have finished reviewing the code, you should perform a series of tests to ensure that it works as intended. These tests should cover a wide range of scenarios, including:

  - Valid and invalid user input
  - Different types of attack vectors
  - Complex state changes

- ``Reporting any problems found``:  If you find any problems with the code, you should report them to the developers of Ethena Labs protocol. The developers will then be able to fix the problems and release a new version of the protocol.

## Codebase quality analysis
There were 6 contracts in scope for this Audit with total nSloc of 588 and 4 external imports. Hence, the code base was pretty small. It was nice to see 70% test coverage for the code as this is not a very complex protocol. But 100% test coverage is recommended.

All the complex functionalities were explained with NatSpec comments and thus, the quality of the code base is pretty good.

This code base also adheres to all the best practices like using name imports, custom errors, indexed event parameters, view and pure functions, etc. Thus, this is a very high quality code base.

## Architecture recommendations
This code base is pretty simple architecture wise as there are not much components to interact for the end user. The architecture is heavily focused for off-chain where on-chain transactions is entirely handled by Ethena.

All the requests needs to be signed off-chain by the user with EIP712 and submitted to Ethena. Ethena then performs series of checks on their end and manually executes the transaction on-chain. 

This was pretty unique but heavily centralized due to lot of roles and access controls. Thus, the chance of rug-pull or grieving by trusted role is high but Ethena is well aware of this issue and thus minimized the losses to 300k per block and necessary freezing mechanisms like the ``GateKeeper`` role.


## Centralization risks

The protocol has a high degree of centralization:

- The Ethena team can freely do minting & redeeming of `USDe` tokens
- The owner of the `StakedUSDe` token contract can manipulate anyone's balance by blacklisting an address and then calling `redistributeLockedAmount` to move his balance to a destination address
- Users can't mint/redeem `USDe` as they only sign transactions which are later executed by Ethena. Ethena does a good amount & variety of off-chain checks on users based on KYC, AML whitelisting and others.

A delegated signer can call `redeem` for a user (if he has approved the `EthenaMinting` contract to burn his `USDe` tokens) and can control the `beneficiary` address who will receive the collateral assets.

## Other recommendations

- Regular code reviews and adherence to best practices
- Conduct external audits by security experts
- Consider open sourcing the contract for community review
- Maintain comprehensive security documentation
- Establish a responsible disclosure policy for vulnerabilities
- Implement continuous monitoring for unusual activity
- Educate users about risks and best practices

## Conclusion
The goal of Ethena is to offer a permissionless stablecoin, USDe, to defi users and to offer users yield for being in the ecosystem. Unlike USDC where Circle captures the yield, USDe holders can stake their USDe in exchange to receive stUSDe, which increases in value relative to USDe as the protocol earns yield. (Similar to rETH increasing in value with respect to ETH)

## Time-spent 

20 Hours 

### Time spent:
20 hours