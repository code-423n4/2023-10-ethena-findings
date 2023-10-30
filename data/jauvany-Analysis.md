**Goals**

The goal of Ethena is to offer a permissionless stablecoin, USDe, to defi users and to offer users yield for being in our ecosystem. Unlike USDC where Circle captures the yield, USDe holders can stake their USDe in exchange to receive stUSDe, which increases in value relative to USDe as the protocol earns yield. (Similar to rETH increasing in value with respect to ETH)

**Scope**

USDe.sol: USDe.sol is the contract of our stablecoin. It extends ERC20Burnable , ERC20Permit and Ownable2Step from Open Zepplin. There's a single variable, the minter address that can be modified by the OWNER . Outside of Ownable2Step contract owner only has one custom function, the ability to set the minter variable to any address

EthenaMinting.sol: EthenaMinting.sol is the contract and address that the minter variable in USDe.sol points to. When users mint USDe with stETH (or other collateral) or redeem collateral for USDe, this contract is invoked.

StackedUSDe.sol: Extension of ERC4626. Users stake USDe to receive stUSDe which increases in value as Ethena deposits protocol yield here

StackedUSDeV2.sol: StakedUSDeV2.sol is where holders of USDe stablecoin can stake their stablecoin, get stUSDe in return, and earn yield. Our protocol's yield is paid out by having a REWARDER role of the staking contract send yield in USDe, increasing the stUSDe value with respect to USDe.

USDeSilo.sol: The Contract to temporarily hold USDe during redemption cooldown.

SingleAdminAccessControl.sol: EthenaMinting uses SingleAdminAccessControl rather than the standard AccessControl.


# Codebase quality analysis
 
> Analysis of the codebase (What’s unique? What’s using existing patterns?): 
 
- Unique: Codebase carries out specific governance mechanisms that are uniquely designed for Ethena’s specific use case e.g. The use of their own USDe stablecoin for Token Transfer.
 
- Existing Patterns: The Ethena Protocol adheres to common contract management patterns, such as the use of onlyRole, and hasRole.
  
**Strengths**
 
- Contract files use fixed compiler versions as recommended.
 
- Rich documentation provided.
 
**Weaknesses**
 
- Named imports of parent contracts are missing
 
- Contract declarations are missing NatSpec @author and @dev tags and functions are missing NatSpec @return, @notice, @param tags.
 
- Function Parameters in Public Accessible Functions Need address(0) Check.
 
- Constructors/Initializers are missing address(0) check.

- The codebase lacks the use of a Modern Upgradeable Contract Paradigm
 
 
# Centralization risks
 
- Ethena contracts have administrator roles that can modify settings if they are able to pause the contracts, there is a risk that these capabilities may be used improperly or become targets for attacks.

# Systemic risks
 
– Like any smart contract-based system, the Ethena Protocol is exposed to potential coding bugs or vulnerabilities. Exploiting these issues could result in the loss of funds or manipulation of the protocol.
 
- Test Coverage: The test coverage provided by The Ethena Protocol is 70%, however, I recommend 100% test coverage.
 
- External Contract Dependencies: The Ethena Protocol relies on OpenZepplin external contracts. If any of these contracts have vulnerabilities, it would affect the protocol.

- Price Manipulation Risk: Since these contracts are designed to calculate the price of USDe, there is a risk that parameters may be manipulated or that input data may be compromised, potentially leading to incorrect prices. 

# Architecture recommendations
 
> The Ethena architecture seems solid in general, none the less Here are some areas that could be improved:

- Testing and Simulations: Even though the Ethena project implements several tests, consider adding more tests to achieve 100% test coverage. I recommend creating a live testnet app. Here is an [example](https://app.opendollar.com/) from The Open Dollar protocol. Conduct thorough testing of all contracts and functions and simulations to understand how they will behave under various market conditions. 

- Improving gas efficiency: Gas can be Optimized by using solidity version 0.8.20 and Optimizer features. New features introduced in Solidity 0.8.20 enhance gas efficiency. Specifically, it takes advantage of the push0 assembler operation for placing 0 on the EVM stack, which reduces both deployment and runtime costs. 


**Other recommendations**
 
- Regular code reviews and adherence to best practices.
- Conduct external audits by security experts.
- Consider open sourcing the contract for community review.
- Maintain comprehensive security documentation.
- Establish a responsible disclosure policy for vulnerabilities.
- Implement continuous monitoring for unusual activity.
- Educate users about risks and best practices.

# Time Spent

> A total of 3 days (24 hours) were dedicated to completing this analysis, distributed as follows: 

– Day 1: I spent time reading the different available documentation in order to have a deep understanding of the protocol.
 
- Day 2: I analyzed the codebase for better understanding and investigated possible systemic risks and centralization risks. 
 
- Day 3: I dedicated this day to coming up with possible Architecture recommendations and preparing the final analysis report.


### Time spent:
24 hours