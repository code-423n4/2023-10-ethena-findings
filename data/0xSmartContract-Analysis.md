# 🛠️ Analysis - Ethena Labs
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" was used  |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Security Approach of the Project | Audit approach of the Project |
|e) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|f) |Packages and Dependencies Analysis | Details about the project Packages |
|g) |Gas Optimization | Gas usage approach of the project and alternative solutions to it |
|h) |Other recommendations | What is unique? How are the existing patterns used? |
|i) |New insights and learning from this audit | Things learned from the project |



## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-10-ethena

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-10-ethena#install)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [Ethena](https://ethena-labs.gitbook.io/ethena-labs/10CaMBZwnrLWSUWzLS2a/) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/crytic/slither)| The project does not currently have a slither result, a slither control was created from initial|
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-10-ethena#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-10-ethena#scope)||
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2023-10-ethena#main-invariants)||

## b) Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.
In this way, many predictions can be made, including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit;
Uses Consensys Solidity Metrics
-  **Lines:** total lines of the source unit
-  **nLines:** normalized lines of the source unit (e.g. normalizes functions spanning multiple lines)
-  **nSLOC:** normalized source lines of code (only source-code lines; no comments, no blank lines)
-  **Comment Lines:** lines containing single or block comments
-  **Complexity Score:** a custom complexity score derived from code statements that are known to introduce code complexity (branches, loops, calls, external interfaces, ...)

![image](https://github.com/code-423n4/2023-10-ethena/assets/104318932/34921fa4-2a43-434d-899e-933e1196b9ac)


</br>
</br>

## EthenaMinting.sol

EthenaMinting.sol is the contract and address that the minter variable in USDe.sol points to. When users mint USDe with stETH (or other collateral) or redeems collateral for USDe, this contract is invoked.

The primary functions used in this contract is mint() and redeem(). Users who call this contract are all within Ethena. When outside users wishes to mint or redeem, they perform an EIP712 signature based on an offchain price we provided to them. 

By design, Ethena will be the only ones calling mint(),redeem() and other functions in this contract.

![image](https://github.com/code-423n4/2023-10-ethena/assets/104318932/537da0d3-8e9f-4bb3-ab79-69bdb237b586)

## StakedUSDe.sol
The StakedUSDe contract allows users to stake USDe tokens and earn a portion of protocol LST and perpetual yield that is allocated to stakers by the Ethena DAO governance voted yield distribution algorithm.  The algorithm seeks to balance the stability of the protocol by funding the protocol's insurance fund, DAO activities, and rewarding stakers with a portion of the protocol's yield.


`transferInRewards()` function allows the owner to transfer rewards from the controller contract into this contract. Therefore, it is an important function and the part whose mechanism needs to be examined the most in the audit;
`transferInRewards()` function:
![image](https://github.com/code-423n4/2023-10-ethena/assets/104318932/8a10b79c-1c22-4574-8f05-2662597bbbb5)

## StakedUSDeV2.sol
The StakedUSDeV2 contract allows users to stake USDe tokens and earn a portion of protocol LST and perpetual yield that is allocate  to stakers by the Ethena DAO governance voted yield distribution algorithm.  The algorithm seeks to balance the stability of the protocol by funding  the protocol's insurance fund, DAO activities, and rewarding stakers with a portion of the protocol's yield.

![image](https://github.com/code-423n4/2023-10-ethena/assets/104318932/28d16221-4e50-42d4-a365-ccdb3c4fd149)


StakedUSDeV2.sol redeem() function;
![image](https://github.com/code-423n4/2023-10-ethena/assets/104318932/307009b1-8e8d-493a-98d9-025fed95ac4a)

StakedUSDeV2.sol unstake() function;
![image](https://github.com/code-423n4/2023-10-ethena/assets/104318932/678e83fd-74ab-4988-8db3-c6859d449764)

StakedUSDeV2.sol withdraw() function;

![image](https://github.com/code-423n4/2023-10-ethena/assets/104318932/8b57a4a0-2873-4f4f-8ca2-7d4feaf2d4e9)

StakedUSDeV2.sol cooldownAssets() function;
![image](https://github.com/code-423n4/2023-10-ethena/assets/104318932/4a363965-2df2-4de3-ab89-93c6f319342b)

StakedUSDeV2.sol cooldownShares() function;
![image](https://github.com/code-423n4/2023-10-ethena/assets/104318932/fbba22db-52a0-4b54-a309-3bc877ae1c93)


## c) Test analysis
### What did the project do differently? ;
-   1) It can be said that the developers of the project did a quality job, there is a test structure consisting of tests with quality content.

-   2) In order to understand the test scenarios and develop more effective test scenarios, the following bob, alice and other roles are defined one by one, the project team has done a good job here;

```solidity
test\utils\LendingMarketHelper.sol:
  58  
  59  
  60:     // labels
  61:     vm.label(bob, 'bob');
  62:     vm.label(alice, 'alice');
  63:     vm.label(DEPLOYER, 'deployer');
  64:     vm.label(USDE_OWNER, 'usde owner');
  65:     vm.label(POOL_PROXY, 'lending pool');
```


### What could they have done better?
- 1) Test suites do not test for re-entrancy
Test teams are testing many functions and variables, but recently, due to the vulnerability in the Vyper Compiler, the hacking of the projects using certain Vyper compiler and losing 50 million $ has revealed the security weakness here. https://cointelegraph.com/news/curve-finance-pools-exploited-over-24-reentrancy-vulnerability

```solidity

contracts\EthenaMinting.sol:
  162:   function mint(Order calldata order, Route calldata route, Signature calldata signature)
  163:     external
  164:     override
  165:     nonReentrant
  166:     onlyRole(MINTER_ROLE)
  167:     belowMaxMintPerBlock(order.usde_amount)
  168:   {


  194:   function redeem(Order calldata order, Signature calldata signature)
  195:     external
  196:     override
  197:     nonReentrant
  198:     onlyRole(REDEEMER_ROLE)
  199:     belowMaxRedeemPerBlock(order.usde_amount)
  200:   {

  247:   function transferToCustody(address wallet, address asset, uint256 amount) external nonReentrant onlyRole(MINTER_ROLE) {


contracts\StakedUSDe.sol:

  89:   function transferInRewards(uint256 amount) external nonReentrant onlyRole(REWARDER_ROLE) notZero(amount) {

  203:   function _deposit(address caller, address receiver, uint256 assets, uint256 shares)
  204:     internal
  205:     override
  206:     nonReentrant
  207:     notZero(assets)
  208:     notZero(shares)
  209:   {

  225:   function _withdraw(address caller, address receiver, address _owner, uint256 assets, uint256 shares)
  226:     internal
  227:     override
  228:     nonReentrant
  229:     notZero(assets)
  230:     notZero(shares)
  231:   {


```
The accuracy of the functions has been tested, but it has not been tested whether the `nonReentrant` modifier in the function works correctly or not. For this, a test must be written that tries to reentrancy and was observed to fail.

Let's take the mint function from the project as an example, we can write any test of this function, it has already been written by the project teams in the test files, but the risk of reentrancy with the reentrant modifier in this function has not been tested, a test file can be added as follows;

Project File:
```solidity

  162:   function mint(Order calldata order, Route calldata route, Signature calldata signature)
  163:     external
  164:     override
  165:     nonReentrant
  166:     onlyRole(MINTER_ROLE)
  167:     belowMaxMintPerBlock(order.usde_amount)
  168:   {

```
</br>

Reentrancy Test  File:
```solidity

// MaliciousContract.sol
pragma solidity ^0.8.0;

import "./YourMainContract.sol";

contract MaliciousContract is YourMainContract {
    function maliciousMint(Order calldata order, Route calldata route, Signature calldata signature) external {
        // Attempt to re-enter the mint function
        this.mint(order, route, signature);
    }
    
    // Override the mint function to include a reentrancy attempt
    function mint(Order calldata order, Route calldata route, Signature calldata signature) external override {
        super.mint(order, route, signature);
        // Reentrancy attempt
        this.mint(order, route, signature);
    }
}



// TestMint.sol
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "./MaliciousContract.sol";

contract TestMint is Test {
    MaliciousContract maliciousContract;

    function setUp() public {
        maliciousContract = new MaliciousContract();
        // Set up any required state or permissions here
    }

    function testNonReentrantModifier() public {
        // Create dummy data for the order, route, and signature
        Order memory order = Order(/*...*/);
        Route memory route = Route(/*...*/);
        Signature memory signature = Signature(/*...*/);

        // Attempt to call the maliciousMint function
        vm.expectRevert("ReentrancyGuard: reentrant call");
        maliciousContract.maliciousMint(order, route, signature);
    }
}

```

</br>





## d) Security Approach of the Project

### Successful current security understanding of the project;

1 - First they did the main audit from Zellic and Quantstamp and resolved all the security concerns in the report 

2- They manage the 2nd audit process with an innovative audit such as Code4rena, in which many auditors examine the codes.


### What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

3- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ; This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

4- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. https://immunefi.com/

5- Pause Mechanism
This is a chaotic situation, which can be thought of as a choice between decentralization and security. Having a pause mechanism makes sense in order not to damage user funds in case of a possible problem in the project.

6- Upgradability
There are use cases of the Upgradable pattern in defi projects using mathematical models, but it is a design and security option.



## e) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2023-10-ethena/blob/main/bot-report.md


**Other Audit Reports (Zellic):**
[ Zellic](https://3400502003-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FsBsPyff5ft3inFy9jyjt%2Fuploads%2FGZbd1DrrG3YmnTlJnTHa%2FEthena%20-%20Zellic%20Audit%20Report%20Draft.pdf?alt=media&token=2e65890c-6724-49b7-97f4-6d2223ba086e)

**Other Audit Reports (Quantstamp):**
[ Quantstamp](https://3400502003-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FsBsPyff5ft3inFy9jyjt%2Fuploads%2F17Ucep7IYMBZ6mAHGLyw%2FEthena%20Final%20Report%20(1).pdf?alt=media&token=51a6a101-516e-4984-8360-14daf860a961)


##  f) Packages and Dependencies Analysis 📦

| Package                                                                                                                                     | Version                                                                                                               | Usage in the project                               | Audit Recommendation                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| [`openzeppelin`](https://www.npmjs.com/package/@openzeppelin/contracts)         | [![npm](https://img.shields.io/npm/v/@openzeppelin/contracts.svg)](https://www.npmjs.com/package/@openzeppelin/contracts)     |               AccessControl.sol,Ownable.sol,Ownable2Step.sol, IERC5313.sol, ReentrancyGuard.sol, ERC20.sol, IERC20.sol, ERC20Burnable.sol, ECDSA.sol,EnumerableSet.sol |-  Version `4.9.2` is used by the project, it is recommended to use the newest version `5.0.0`                                                                                     |



## g) Gas Optimizations
When the project is analyzed in terms of Gas Optimization, there is a very important gas optimization;
"Using Mapping instead of Openzeppelin's EnumerableSet library provides high gas optimization"


Other than that, there are some functions written in inline assembly or very minor gas optimizations but they are not mentioned because their effect is very very low, all possible minor gas optimizations are already listed in Auto Bot;
https://github.com/code-423n4/2023-10-ethena/blob/main/bot-report.md#gas-findings

### Using Mapping instead of Openzeppelin's EnumerableSet library provides high gas optimization

The project used Openzeppelin's EnumerableSet library in the following parts;

```solidity
contracts\EthenaMinting.sol:
  12: import "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

  23:   using EnumerableSet for EnumerableSet.AddressSet;

  65    /// @notice Supported assets
  66:   EnumerableSet.AddressSet internal _supportedAssets;

  68    // @notice custodian addresses
  69:   EnumerableSet.AddressSet internal _custodianAddresses;
```




The use of `EnumerableSet` takes place in the following steps;
1- Define Openzeppelin library with `import`
2- With `using EnumerableSet for EnumerableSet.AddressSet;` it can be used directly without specifying the library name with its definition.
3- In `EnumerableSet.AddressSet private s_priceUpdaters;` a private s_priceUpdaters type variable is declared using the EnumerableSet library

In the following 2 state variables for Gas Optimization, using mapping instead of EnumerableSet will save gas;

```solidity
contracts\EthenaMinting.sol:
  66:   EnumerableSet.AddressSet internal _supportedAssets;
  69:   EnumerableSet.AddressSet internal _custodianAddresses;

```

EnumerableSet is offers efficient operations for adding, removing, and checking the existence of elements in the set.

Here are some key features of EnumerableSet:

Constant-Time Operations: The operations of adding, removing, and checking for existence in a set using EnumerableSet are performed in constant time, which means they have a fixed gas cost regardless of the size of the set. This makes it a suitable choice for scenarios where fast and efficient lookup is required.

Enumeration Capability: EnumerableSet allows you to iterate over the elements in the set. While sets do not guarantee any specific ordering of elements, you can efficiently enumerate through the set using functions like length(), at(index), etc. This enables you to perform operations on each element or retrieve specific elements from the set.

But using Mapping in this project seems more efficient

Because while EnumerableSet provides enumeration capabilities, iterating over a large set consumes a significant amount of gas, especially if you need to perform operations on each item, which is exactly what happens in this project. It's more efficient to use a mapping if accessing them by key has a fixed gas cost

Gas optimization could not be measured due to the complete change of the architecture, but it is obvious that it will provide a significant gas gain in the context of use cases.

## h) Other recommendations

✅ The use of assembly in project codes is very low, I especially recommend using such useful and gas-optimized code patterns; https://github.com/dragonfly-xyz/useful-solidity-patterns/tree/main/patterns/assembly-tricks-1

✅ It is seen that the latest versions of imported important libraries such as Openzeppelin are not used in the project codes, it should be noted. https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/

✅A good model can be used to systematically assess the risk of the project, for example this modeling is recommended; https://www.notion.so/Smart-Contract-Risk-Assessment-3b067bc099ce4c31a35ef28b011b92ff#7770b3b385444779bf11e677f16e101e






## i) New insights and learning from this audit 

🔎 1. Understanding Ethena’s USDe Ecosystem

Ethena aims to provide a permissionless stablecoin, USDe, to decentralized finance (defi) users, while also offering yield for participating in its ecosystem. This model differs from USDC as it allows USDe holders to stake their stablecoin to receive stUSDe, a token that appreciates in value as the protocol generates yield, similar to the relationship between rETH and ETH.

🔎 2. Minting USDe through Ethena

USDe is minted by trading stETH based on a quoted rate provided by Ethena. Users sign an EIP712 signature to confirm the transaction, after which stETH is traded for USDe, and the stETH is then used to create a delta neutral position through custodians and a perpetual exchange.

🔎 3. Ethena's Yield Generation Mechanism

Users mint USDe using stETH, and Ethena opens a corresponding short position in ETH perps. The yield from stETH and the short ETH perps position are combined and sent to an insurance fund, and subsequently distributed to the staking contract every 8 hours.

🔎 4. Delta Neutrality in Ethena’s Ecosystem

Ethena maintains a delta neutral position through long stETH and short ETH perps positions. This ensures that the value of the position remains fixed, regardless of market fluctuations, providing security to users wishing to redeem their USDe.

🔎 5. Redeeming USDe in Ethena’s System

 In the event of a significant market downturn, Ethena’s delta neutral position allows users to redeem their USDe at an equivalent value, by closing the short perps position to realize profits and using those to buy back the necessary amount of stETH to fulfill the redemption. This ensures that users receive back the full value of their initial investment, adjusted for market conditions.


### Time spent:
15 hours