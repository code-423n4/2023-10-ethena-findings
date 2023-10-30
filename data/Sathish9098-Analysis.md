# Ethena Labs Analysis

## Overview
- Permissionless stablecoin, USDe
- USDe holders can stake their USDe in exchange for yield
- USDe is minted by depositing stETH and shorting ETH perps
- USDe is backed by a delta-neutral position of stETH and short ETH perps

## Approach taken in evaluating the codebase
In the initial phase of our code review, conducted a thorough assessment of the project's scope. This assessment provided us with a clear understanding of the project's objectives, architecture, and the specific components that needed to be reviewed.

### USDe Contract
The USDe contract is responsible for the Ethena stablecoin. Only the minter address can mint USDe, and this address is always pointed to the EthenaMinting.sol contract.

### EthenaMinting Contract
The EthenaMinting contract is responsible for minting and redeeming USDe. It is only called by the Ethena backend

### StakedUSDeV2 Contract
The StakedUSDeV2 contract allows users to stake their USDe and earn yield. There are some restrictions on who can use the staking contract and how they can use it, but these restrictions are in place to protect the protocol and its users.

## Architecture Overview

### ```EthenaMinting Contract```

### Minting Flow
Validates and executes a mint order, transferring collateral from the benefactor to the Ethena protocol and minting USDe tokens for the beneficiary.

```solidity
function mint(Order calldata order, Route calldata route, Signature calldata signature)
    external
    override
    nonReentrant
    onlyRole(MINTER_ROLE)
    belowMaxMintPerBlock(order.usde_amount)
  {
    if (order.order_type != OrderType.MINT) revert InvalidOrder();
    verifyOrder(order, signature);
    if (!verifyRoute(route, order.order_type)) revert InvalidRoute();
    if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();
    // Add to the minted amount in this block
    mintedPerBlock[block.number] += order.usde_amount;
    _transferCollateral(
      order.collateral_amount, order.collateral_asset, order.benefactor, route.addresses, route.ratios
    );
    usde.mint(order.beneficiary, order.usde_amount);
    emit Mint(
      msg.sender,
      order.benefactor,
      order.beneficiary,
      order.collateral_asset,
      order.collateral_amount,
      order.usde_amount
    );
  }

```
![Minting](https://user-images.githubusercontent.com/58845085/279150475-1b308cf1-2bf9-4bb4-b3d9-d7e5f7ebe1b4.PNG)


### Redeem Flow
Validates and executes a redeem order, burning USDe tokens from the benefactor and transferring collateral to the beneficiary

```solidity

function redeem(Order calldata order, Signature calldata signature)
    external
    override
    nonReentrant
    onlyRole(REDEEMER_ROLE)
    belowMaxRedeemPerBlock(order.usde_amount)
  {
    if (order.order_type != OrderType.REDEEM) revert InvalidOrder();
    verifyOrder(order, signature);
    if (!_deduplicateOrder(order.benefactor, order.nonce)) revert Duplicate();
    // Add to the redeemed amount in this block
    redeemedPerBlock[block.number] += order.usde_amount;
    usde.burnFrom(order.benefactor, order.usde_amount);
    _transferToBeneficiary(order.beneficiary, order.collateral_asset, order.collateral_amount);
    emit Redeem(
      msg.sender,
      order.benefactor,
      order.beneficiary,
      order.collateral_asset,
      order.collateral_amount,
      order.usde_amount
    );
  }

```

![Redeem](https://user-images.githubusercontent.com/58845085/279151485-8c665f49-09c9-4742-a14c-0f66cc10c0c9.PNG)


### Gnosis Safe Multisig

Gnosis Multisig is a multi-signature wallet that allows multiple users to sign and approve transactions before they are executed. This makes it a more secure and reliable way to manage funds and assets, especially for large amounts or high-value transactions

Gnosis Multisig proxy architecture consists of the following components:

- ``Gnosis Safe``: The Gnosis Safe is the main contract that provides the functionality of the multisig wallet. It is responsible for storing the wallet's assets and executing transactions.
- ``Gnosis SafeProxy``: The Gnosis SafeProxy is a proxy contract that sits between the Gnosis Safe and the Ethereum blockchain. It is responsible for forwarding transactions to the Gnosis Safe for execution.
- ``Gnosis SafeProxyFactory``: The Gnosis SafeProxyFactory is a contract that is used to create new Gnosis SafeProxy contracts.

Using a Gnosis multisig to hold ownership of the smart contracts is a good security practice. A multisig wallet requires multiple signatures to approve transactions, which makes it more difficult for hackers to steal funds or otherwise disrupt the protocol.

It is also good that Ethena is using cold wallets for the multisig keys. Cold wallets are not connected to the internet, which makes them much more difficult to hack.

Requiring 7/10 or more confirmations before transactions are approved is a good way to ensure that the multisig is used securely. This means that at least 7 of the 10 multisig key holders must approve a transaction before it can be executed. This helps to protect against the possibility of a single key holder being compromised.

Overall, using a Gnosis multisig with cold wallets and 7/10 confirmations is a good way to secure the ownership of Ethena's smart contracts.

#### Possible Risks Gnosis Safe Multisig Contracts

- ``Slow decision-making``
- If it takes a long time to get 7/10 multisig key holders to agree on a transaction, this could slow down the development and deployment of new features on Ethena.
 - ``Centralization`` 
If a small number of key holders control a large majority of the multisig keys, this could lead to centralization of power. This could make it easier for a small group of people to control Ethena and its future direction.
- ``Lockup risk``
-  If a multisig key holder loses their key or becomes unavailable, this could lock up funds or prevent Ethena from making important changes.

> Mitigation Suggestions

- Providing a clear and transparent process for multisig key holders to vote on proposals
- Having a backup plan in place in case a multisig key holder is unable to access their key

## Architecture Recommendations

### Use a more secure multisig wallet
The Gnosis Safe multisig wallet is a good choice, but there are other options that offer additional security features, such as hardware-based signing and multi-factor authentication.

### Consider using a trusted execution environment (TEE)
A TEE is a secure environment within a processor that can be used to isolate and protect sensitive data. Using a TEE could help to protect the Ethena multisig keys from compromise.

### Consider using a decentralized oracle
The Ethena smart contracts currently rely on a centralized oracle to provide market prices. Using a decentralized oracle would make the protocol more resistant to manipulation

### Implement a circuit breaker
A circuit breaker is a mechanism that can be used to temporarily disable minting and redeems if there is a sharp movement in the price of ETH or stETH. This would help to prevent the Ethena protocol from being liquidated

### Partner with other DeFi protocols
Partnering with other DeFi protocols could help to increase the adoption of Ethena and provide users with more options for interacting with the protocol

## Codebase quality analysis

- Use Safe Math for all arithmetic operations involving ``mintedPerBlock``, ``redeemedPerBlock``, ``maxMintPerBlock``, and ``maxRedeemPerBlock``. SafeMath can be used for extra safety, even with Solidity 0.8.0 and later, to make the code more explicit about its intentions regarding avoiding overflows and underflows.
- Use function ``modifiers`` for repetitive ``access control checks`` to reduce ``code duplication`` and ``improve readability``
- Include more data in emitted ``events`` to provide greater transparency and context for ``external systems`` and ``users``. This can help with ``tracking`` and ``auditing``
- Ensure that input ``validation checks`` cover all possible edge cases and scenarios, reducing the risk of unexpected behavior
- Consider breaking down ``complex functions`` into ``smaller``, more ``modular functions`` with well-defined purposes. This can enhance ``code readability`` and ``maintainability``
- Maintain consistent ``naming conventions`` for variables and functions. Make sure all names are ``self-descriptive`` 

## Systemic Risks

Here's an analysis of potential systemic and centralization risks in the contract:

### Custodian risk
Ethena delegates the custody of its collateral to ``third-party custodians`` in order to achieve ``scalability`` and ``security``. This means that Ethena does not hold the ``collateral itself``, but instead relies on the ``custodians`` to keep it safe and secure

#### Custodian failure could lead to the collapse of the Ethena stablecoin peg

- ``If a custodian is hacked, the hackers could steal the collateral that Ethena has deposited with them``- This would leave Ethena with insufficient collateral to back its USDe tokens, and the stablecoin peg would collapse.
- ``If a custodian goes bankrupt, it may not be able to return the collateral to Ethena``-  This would also leave Ethena with insufficient collateral to back its USDe tokens, and the stablecoin peg would collapse.
- ``If a custodian is subject to a regulatory crackdown, it may be forced to liquidate Ethena's collateral``- This would also leave Ethena with insufficient collateral to back its USDe tokens, and the stablecoin peg would collapse.

> Mitigation Suggestions

- Using reputable custodians
- Diversifying custodian holdings

### Counterparty risk
Counterparty risk is the risk that one party to a financial contract will not fulfill its obligations. In the case of Ethena, the counterparty risk is the risk that the perps exchange that Ethena uses to generate yield on its collateral will become insolvent or default on its obligations

If this happens, Ethena could lose its collateral and be unable to mint new USDe tokens or redeem existing USDe tokens. This could have a number of negative consequences for Ethena users, including:

- A loss of funds, if Ethena is unable to redeem USDe tokens for their underlying collateral.
- A decrease in the value of USDe tokens, as the market would lose confidence in Ethena's ability to maintain the stablecoin peg.
- An increase in the cost of borrowing USDe tokens, as lenders would demand a higher risk premium to compensate for the increased counterparty risk.

> Mitigation Suggestions

- Monitoring perps exchange health

### Oracle risk

Oracle risk is the risk that the price data provided by oracles is inaccurate or manipulated. In the case of Ethena, the oracle risk is the risk that the oracles that Ethena relies on to provide price data for ETH and stETH are compromised or malfunction.

If this happens, Ethena could misprice USDe and StakedUSDeV2. This could lead to a number of negative consequences for Ethena users, including:

- Losses for users who mint or redeem USDe tokens at an inaccurate price.
- Losses for users who stake or unstake USDe tokens for stUSDe tokens at an inaccurate price.
- Reduced liquidity for USDe and StakedUSDeV2 tokens, as traders may be reluctant to trade tokens that are priced inaccurately.

> Mitigation Suggestions

- Using multiple oracles
- Monitoring oracle performance
- Using decentralized oracles

### Market risk
Sharp movements in the price of ETH or stETH could lead to losses for Ethena users. This is because Ethena maintains a delta-neutral position between stETH and short ETH perps. This means that Ethena buys stETH and shorts ETH perps in order to hedge its exposure to the price of ETH.

If the price of ETH or stETH moves sharply, Ethena may not be able to adjust its positions quickly enough. This could lead to Ethena being liquidated on its short ETH perps position or to Ethena losing money on its stETH holdings.

### Smart contract vulnerabilities
Ethena relies on a number of smart contracts, including ``USDe.sol``, ``EthenaMinting.sol``, and ``StakedUSDeV2.sol``. If there are any vulnerabilities in these contracts, they could be exploited by attackers to steal funds or otherwise disrupt the Ethena ecosystem.

### Dependency risk

- We observed that old and unsafe versions of Openzeppelin are used in the project, this should be updated to the latest one:

```
// OpenZeppelin Contracts (last updated v4.9.0) (token/ERC20/ERC20.sol)
```
- Latest is 4.9.3 (as of July 28, 2023), version used in project is 4.9.0

## Centralization Risks

There are several important roles in the system.

### Privileged Roles

#### MINTER_ROLE 

```solidity
bytes32 private constant MINTER_ROLE = keccak256("MINTER_ROLE");
```
Only addresses that have been assigned the MINTER_ROLE can successfully call the ``mint`` and ``transferToCustody`` functions.

#### REDEEMER_ROLE

```solidity
bytes32 private constant REDEEMER_ROLE = keccak256("REDEEMER_ROLE");
```
Only addresses that have been assigned the MINTER_ROLE can successfully call the ``redeem()`` function.

####  GATEKEEPER_ROLE

```solidity
bytes32 private constant GATEKEEPER_ROLE = keccak256("GATEKEEPER_ROLE");

```
Only addresses that have been assigned the MINTER_ROLE can successfully call the ``disableMintRedeem()``,``removeMinterRole()``,``removeRedeemerRole()`` functions.

It is also good that Ethena is using cold wallets for the ``multisig keys``. Cold wallets are not connected to the internet, which makes them much more difficult to hack.

Requiring ``7/10`` or more confirmations before transactions are approved is a good way to ensure that the ``multisig ``is used ``securely``. This means that at least 7 of the 10 multisig key holders must approve a transaction before it can be executed. This helps to protect against the possibility of a single key holder being compromised.

## Other Recommendations 

### Test Coverage
```
What is the overall line coverage percentage provided by your tests?: 70%
```
The audit scope of the contracts should be the aim of reaching 100% test coverage. However, the tests provided by the protocol are at a high level of comprehension, giving us the opportunity to easily conduct Proof of Concepts (PoCs) to test certain bugs present in the project.

### Suggests to add tests attack vectors, especially re-entrancy

```solidity

function redeem(Order calldata order, Signature calldata signature)
    external
    override
    nonReentrant
    onlyRole(REDEEMER_ROLE)
    belowMaxRedeemPerBlock(order.usde_amount)
    
function mint(Order calldata order, Route calldata route, Signature calldata signature)
    external
    override
    nonReentrant
    onlyRole(MINTER_ROLE)
    belowMaxMintPerBlock(order.usde_amount)
    
function transferToCustody(address wallet, address asset, uint256 amount) external nonReentrant onlyRole(MINTER_ROLE) {

```
The impact of a reentrancy vulnerability can be severe. An attacker can drain funds from the vulnerable contract, manipulate its state, or even render it unusable. Even reentrancy guard added should be tested.

Attack vectors, especially re-entrancy, seem untested and trusted.

## New insights and learning from this audit

- One of the key technical learnings from Ethena is the use of a delta-neutral strategy to maintain the stability of USDe
- Another key technical learning from Ethena is the use of a gnosis safe multisig to hold ownership of its smart contracts
- Ethena's staking contract also incorporates a number of technical innovations, such as a vesting period to prevent frontrunning and on-chain mint and redeem limitations to prevent malicious actors from manipulating the market
- Ethena has GATEKEEPER roles that can disable minting and redeems and remove MINTER and REDEEMER roles, as well as SOFT-RESTRICTED-STAKER and FULL-RESTRICTED-STAKER roles for addresses based in countries that Ethena is not allowed to provide yield to and sanction/stolen funds

## Time spent
10 Hours













### Time spent:
10 hours